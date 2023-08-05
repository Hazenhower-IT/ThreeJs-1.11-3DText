<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted} from "vue"
import { OrbitControls} from "three/examples/jsm/controls/OrbitControls"
import {FontLoader} from "three/examples/jsm/loaders/FontLoader"
import {TextGeometry} from "three/examples/jsm/geometries/TextGeometry"
import * as THREE from "three"

var controls


let canvasRef = ref();

//Utilizziamo i valori del viewport per impostare la grandezza del canvas
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}

//Aggiungiamo l'event listner per il resize del canvas
window.addEventListener("resize", ()=>{
  //update sizes for the canvas
  sizes.width = window.innerWidth
  sizes.height = window.innerHeight

  //update the camera aspect ratio
  camera.aspect = sizes.width / sizes.height
  camera.updateProjectionMatrix()

  //update renderer
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
})


//HANDLE FULL SCREEN (Il codice aggiuntivo è per permettere il funzionamento corretto anche con browser Safari)
window.addEventListener("dblclick",()=>{

  const fullscreenElement = document.fullscreenElement ||  document.webkitFullscreenElement
  if(!fullscreenElement){
    if(canvasRef.value.requestFullscreen){
      canvasRef.value.requestFullscreen()
    }
    else if(canvasRef.value.webkitRequestFullscreen){
      canvasRef.value.webkitRequestFullscreen()
    }
    
  }
  else{
    if(document.exitFullscreen)
    {
      document.exitFullscreen()
    }
    else if(document.webkitExitFullscreen){
      document.webkitExitFullscreen()
    }
  }
})

let scene = new THREE.Scene()

let renderer

//FONTLOADER & 3D TEXT 

//for use our/custom font in threeJs , we must convert it from fonts to typeface format, you can do that using this website:  https://gero3.github.io/facetype.js/

//then load the typecas font using the Appropriate loader

//NOTE  you can import the font also using => import fontName from "the/font/folder/path"

const fontLoader = new FontLoader()
const textureLoader = new THREE.TextureLoader()

const matcapTexture = textureLoader.load("/textures/matcaps/3.png")

fontLoader.load(
  "/fonts/helvetiker_bold.typeface.json",
  (font)=>{
    const textGeometry = new TextGeometry("StreamSystemDesigner",
    {
      font: font,
      size: 0.5,
      height: 0.2,
      curveSegments: 4,
      bevelEnabled: true,
      bevelThickness: 0.03,
      bevelSize: 0.02,
      bevelOffset: 0,
      bevelSegments: 5
    })
    //CENTER THE TEXT
    // By default the center of the text is attached to the bottom-left side of the text
    //We want to be able to center the text so we can translate and rotate the text correctly, and we have 2 solution for doing that:

    //Using Bounding
    //bounding is an information associated with the geometry that tells what space is taken by that geometry, it can be a box or a sphere
    //it helps threejs calculate if the object is on the screen (frustrum culling)

    //NOTE threejs by default is using a sphere bounding
    /*
    //but we can calculate the box bounding using this method 
    
    textGeometry.computeBoundingBox()
    console.log(textGeometry.boundingBox)

    //so to rotate correctly we need to translate the geometry(not the mesh) of half oh the boundingbox width, so we have centered the text
    
    textGeometry.translate(
      -textGeometry.boundingBox.max.x * 0.5,
      -textGeometry.boundingBox.max.y * 0.5,
      -textGeometry.boundingBox.max.z * 0.5
    )

    * */


    //There is another way to center the text, that is much faster and simple. just use this method
    textGeometry.center()

    const textMaterial = new THREE.MeshMatcapMaterial({matcap: matcapTexture})
    const text = new THREE.Mesh(
      textGeometry, textMaterial
    )
    scene.add(text)


    console.time("donuts")

    const donutGeo = new THREE.TorusBufferGeometry(0.3,0.2,20,45)
    //const donutMat = new THREE.MeshMatcapMaterial({matcap: matcapTexture})
     

    for(let i=0; i<100; i++){
      
      const donut = new THREE.Mesh(donutGeo, textMaterial)

      donut.position.x = (Math.random() - 0.5) * 25
      donut.position.y = (Math.random() - 0.5) * 25
      donut.position.z = (Math.random() - 0.5) * 25
      
      donut.rotation.x = Math.random() * Math.PI
      donut.rotation.z = Math.random() * Math.PI

      const donutScale = Math.random()
      donut.scale.set(donutScale, donutScale, donutScale)

      scene.add(donut)
    }
    console.timeEnd("donuts")
  }
)

//Create a Text Mesh is an hard work for a computer, so try to avoid performance issue by reducing the curveSegments and bevelSegments as mush as possible without reducing too much the detail




//the text is not centered so we create a new axis helper to help us doing that
const axesHelper = new THREE.AxesHelper()
scene.add(axesHelper)



// CAMERA
let camera = new THREE.PerspectiveCamera(
  75, 
  sizes.width / sizes.height, 
  0.1,
  100  
);

camera.position.set(0,0,3)
scene.add(camera);

const clock = new THREE.Clock()

const animate = () =>{
  const elapsedTime = clock.getElapsedTime()

  controls.update()
  
  renderer.render(scene, camera)
}

onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value
  });
 
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  //è possivile che veriamo un po di imperfezioni sul render, soprattutto sugli angoli
  //il pixel ratio corrisponde a quanti pixel fisici hai sul tuo schermo, per un unita di pixel del software
  renderer.setPixelRatio(Math.min(window.devicePixelRatio,2)) // in questo modo il max valore di pixel ratio utilizzaro è 2 (altrimenti devi renderizzare troppi pixel, dispiego enorme di potenza gpu)
  renderer.render(scene, camera)
  renderer.setAnimationLoop(animate)

  

  //CONTROLS
  controls = new OrbitControls(camera, canvasRef.value)
  controls.enableDamping = true; // permette uno effetto smooth una volta che rilasciamo l'elemento, rallenta fino a fermarsi
});

onUnmounted(() => {
  renderer.setAnimationLoop(null)
});

</script>

<style>
canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>