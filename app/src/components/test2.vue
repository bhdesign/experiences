<script lang="ts">
import { defineComponent,ref,onMounted,onUnmounted } from 'vue'

export default defineComponent({
  name: 'test2',
})
</script>
<script setup lang="ts">
import{
    Scene,
BoxGeometry,
MeshStandardMaterial,
PerspectiveCamera,
WebGLRenderer,
Mesh
} from 'three'
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js'
let scene:Scene,camera:PerspectiveCamera,renderer:WebGLRenderer, controls:OrbitControls,amount=2;
const canvas=ref()
let isRunning=ref(true);
const fshader = `
void main()
{
  vec3 color = vec3(0.5);
  gl_FragColor = vec4(color, 1.0);
}
`
function initScene(){
   scene = new Scene();
 camera = new PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    1,
    1000
  );
camera.position.z = 100;

 renderer = new WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
canvas.value.appendChild( renderer.domElement );

const geometry = new BoxGeometry( 30, 30, 30, 10, 10, 10 );
const material = new MeshStandardMaterial( {
  wireframe: true
} );
material.onBeforeCompile=(shader)=>{ // beforecompile before the shader is compiled we inject modification
  
  shader.uniforms.time = { value: 0};
  shader.uniforms.radius = { value: 30 };
  shader.vertexShader = 'uniform float time;\n uniform float radius;\n' +shader.vertexShader;
  shader.vertexShader = shader.vertexShader.replace(
						`#include <begin_vertex>`,
            `#include <begin_vertex>
              transformed /= radius;       
              transformed += vec3( 0.5 * ( 1.0 + sin(  1.-time  ) ) * ( normalize( transformed ) - transformed ) );
              transformed *= radius;
            `
          );
 
  shader.fragmentShader=fshader;
  material.userData.shader = shader;
}
material.customProgramCacheKey = function () {

return amount.toFixed( 1 );

};

const ball = new Mesh( geometry, material );
scene.add( ball );

controls = new OrbitControls(camera, renderer.domElement);

window.addEventListener( 'resize', onWindowResize, false );

animate();

}

function onWindowResize() {
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize( window.innerWidth, window.innerHeight );
}


function animate() {
  if(isRunning.value){
    requestAnimationFrame( animate );
  controls.update();
  scene.traverse( function ( child ) {
    if ( (child as Mesh).isMesh ) {
  const shader = ((child as Mesh).material as MeshStandardMaterial).userData.shader;
  if ( shader ) {
    shader.uniforms.time.value = performance.now() / 1000;
  }
}
});
  renderer.render( scene, camera );
  }
  
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
<div ref="canvas"></div>
</template>