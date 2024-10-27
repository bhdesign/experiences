<script lang="ts">
import { defineComponent,ref,onMounted,onUnmounted } from 'vue'

export default defineComponent({
  name: 'test3',
})
</script>
<script setup lang="ts">
import{
    Scene,
    PlaneGeometry,
    ShaderMaterial,
    OrthographicCamera,
WebGLRenderer,
Color,Clock,
Mesh
} from 'three'
let scene:Scene,camera:OrthographicCamera,renderer:WebGLRenderer,clock:Clock
const canvas=ref()
let isRunning=ref(true);
const vshader = `
varying vec2 v_uv;
void main() {	
  v_uv = uv;
  gl_Position = projectionMatrix * modelViewMatrix * vec4( position, 1.0 );
}
`
const fshader = `
varying vec2 v_uv;

uniform vec3 u_color_a;
uniform vec3 u_color_b;
uniform vec2 u_resolution;
uniform float u_time;
uniform vec2 u_mouse;

/***Don't touch***/
float line(float x, float y, float line_width, float edge_width){
  return smoothstep(x-line_width/2.0-edge_width, x-line_width/2.0, y) - smoothstep(x+line_width/2.0, x+line_width/2.0+edge_width, y);
}

/***Override this method with the solution and use it on the main function***/
float brick(vec2 pt, float mortar_height, float edge_thickness){
    float brickWidth = pt.x;
    float brickHeight = pt.y;
    float mortarThickness = mortar_height;
    float edgeWidth = edge_thickness;
    //vec2 pos = v_uv*( ( 1.0 + sin(  1.-u_time  )*0.05 ));
    vec2 pos = v_uv;
    pos.y /= brickHeight;
    //pos.x -= mod(floor(pos.y), 2.) * 0.5 * brickWidth*u_time; // animate row
    pos.x -= mod(floor(pos.y), 2.) * 0.5 * brickWidth;
    pos.x /= brickWidth;
   
    vec2 brickCoord = fract(pos);
    bool isMortar = brickCoord.x < mortarThickness / brickWidth || brickCoord.y < mortarThickness / brickHeight;
    float horizontalLine = line(0.5, fract(pos.y), float(isMortar),mortarThickness * brickHeight);
    float verticalLine = line(0.5, fract(pos.x), float(isMortar),mortarThickness * brickWidth);
    float mortar = min(horizontalLine, verticalLine);
    return mortar;
}

void main (void)
{
    float color=brick(vec2(0.25,0.125),0.01,0.005)*u_time;
  gl_FragColor = vec4(mix(u_color_a, u_color_b, color), 1.0);
}
`
const uniforms = {
  u_color_a: { value: new Color(0x0000ff) },
  u_color_b: { value: new Color(0xffffff) },
  u_time: { value: 0.0 },
  u_mouse: { value:{ x:0.0, y:0.0 }},
  u_resolution: { value:{ x:window.innerWidth, y:window.innerHeight }}
}
function initScene(){
   scene = new Scene();
 camera =new  OrthographicCamera( -1, 1, 1, -1, 0.1, 10 );

 renderer = new WebGLRenderer();
renderer.setSize( window.innerWidth, window.innerHeight );
canvas.value.appendChild( renderer.domElement );
clock = new Clock();
const geometry = new PlaneGeometry( 2, 2 );

const material = new ShaderMaterial( {
  uniforms: uniforms,
  vertexShader: vshader,
  fragmentShader: fshader
} );
const plane = new Mesh( geometry, material );
scene.add( plane );

camera.position.z = 1;
if ('ontouchstart' in window){
  document.addEventListener('touchmove', move);
}else{
  window.addEventListener( 'resize', onWindowResize, false );
  document.addEventListener('mousemove', move);
}
animate();
}

function onWindowResize() {
  const aspectRatio = window.innerWidth/window.innerHeight;
  let width, height;
  if (aspectRatio>=1){
    width = 1;
    height = (window.innerHeight/window.innerWidth) * width;
    
  }else{
    width = aspectRatio;
    height = 1;
  }
  camera.left = -width;
  camera.right = width;
  camera.top = height;
  camera.bottom = -height;
  camera.updateProjectionMatrix();
  renderer.setSize( window.innerWidth, window.innerHeight );
  uniforms.u_resolution.value.x = window.innerWidth;
  uniforms.u_resolution.value.y = window.innerHeight;
}
function move(evt:Event){
    if ('touches' in evt) {
    const touchEvent = evt as TouchEvent;
    uniforms.u_mouse.value.x = touchEvent.touches[0].clientX;
    uniforms.u_mouse.value.y = touchEvent.touches[0].clientY;
  } else {
    const mouseEvent = evt as MouseEvent;
    uniforms.u_mouse.value.x = mouseEvent.clientX;
    uniforms.u_mouse.value.y = mouseEvent.clientY;
  }
}

function animate() {
  if(isRunning.value){
    requestAnimationFrame( animate );
  uniforms.u_time.value += clock.getDelta();
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

	if ((obj.material as ShaderMaterial).isMaterial) {
		(obj.material as ShaderMaterial).dispose()
	} else {
		if (Array.isArray(obj.material)) {
    // an array of materials
    for (const material of obj.material) {
        material.dispose();
    }
} else {
    // single material
    (obj.material as ShaderMaterial).dispose();
}
	}
})
  isRunning.value=false
 })
</script>
<template>
<div ref="canvas"></div>
</template>