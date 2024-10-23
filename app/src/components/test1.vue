<script lang="ts">
import { defineComponent,ref,onMounted, onUnmounted } from 'vue'

export default defineComponent({
  name: 'test1',
})
</script>
<script setup lang="ts">
import{
    Scene,
    IcosahedronGeometry,
    MeshStandardMaterial,
    ShaderMaterial,
BufferAttribute,
PerspectiveCamera,
WebGLRenderer,
Mesh,
AmbientLight,
PointLight,
DoubleSide,
TypedArray
} from 'three'

let scene:Scene,camera:PerspectiveCamera,renderer:WebGLRenderer,ico:Mesh,materialWire:ShaderMaterial, material:MeshStandardMaterial, isSwap=ref(false)
const canvas=ref();
let isRunning=ref(true)
const isvertexShader = `
  attribute vec3 a_barycentric;
  varying vec3 vbc;
    
  void main() {
    vbc = a_barycentric;
    gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);
  }
`;

const isfragmentShader = `
  precision mediump float;
  varying vec3 vbc;
    uniform vec3 color;
  const float lineWidth = 1.0;

  float edgeFactor() {
    vec3 d = fwidth(vbc);
    vec3 f = step(d * lineWidth, vbc);
    return min(min(f.x, f.y), f.z);
  }

  void main() {
    gl_FragColor = vec4(mix(vec3(0,1,0), vec3(0),edgeFactor()), 1.0);
  }
`;

function initScene(){
    scene = new Scene();
 camera = new PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 0.1, 1000 );
camera.position.z = 30;

renderer = new WebGLRenderer({antialias:true});
renderer.setSize( window.innerWidth, window.innerHeight );
canvas.value.appendChild( renderer.domElement );
material = new MeshStandardMaterial( {color: 0x00ff00} ); 
materialWire = new ShaderMaterial({
    vertexShader:isvertexShader,
          fragmentShader:isfragmentShader,
           transparent:false,
           side:DoubleSide
        });
        
const geometry2 = new IcosahedronGeometry(8,1);
const barycentric=calculateBarycentric(geometry2.attributes.position.array);
geometry2.computeVertexNormals();
geometry2.setAttribute('a_barycentric', new BufferAttribute(barycentric, 3));
ico = new Mesh(geometry2, material);
scene.add( ico );
scene.add( camera );

//light setup
const light = new AmbientLight( 0x404040 ); 
scene.add( light );
const light1 = new PointLight( 0xffffff, 100 );
scene.add(light1);
light1.position.y+=10;
light1.position.z+=25;
animate();

window.addEventListener( 'resize', onWindowResize, false );
}

function animate() {
  if(isRunning.value){
  ico.rotation.x += .005;
  ico.rotation.y += .005;
  ico.rotation.z += .005;
  requestAnimationFrame( animate );
  renderer.render( scene, camera );
}

}
function onWindowResize(){
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    
    renderer.setSize( window.innerWidth, window.innerHeight );
}
function swapMat(){
    isSwap.value=!isSwap.value;
    isSwap.value? ico.material=materialWire:ico.material=material;
}
function calculateBarycentric(ar:TypedArray){
   
  const n = ar.length / 6;
  const barycentric = []
  for (let i = 0; i < n; i++) barycentric.push(1, 0, 0, 0, 1, 0, 0, 0, 1)
  return new Float32Array(barycentric)
}

onMounted(()=>{
    initScene()
})
 onUnmounted(()=>{
  renderer.dispose()

scene.traverse(object => {
  let obj=object as Mesh;
	if (!obj.isMesh) return
	
	console.log('dispose geometry!')
	obj.geometry.dispose()

	if ((obj.material as MeshStandardMaterial).isMaterial) {
		(obj.material as MeshStandardMaterial).dispose()
	} else {
		if (Array.isArray(obj.material)) {
    // an array of materials
    for (const material of obj.material) {
        material.dispose();
    }
} else {
    // single material
    (obj.material as MeshStandardMaterial).dispose();
}
	}
})
  isRunning.value=false
 })
</script>

<template>
<button @click.prevent="swapMat()">Swap</button>
<div ref="canvas"></div>
</template>
