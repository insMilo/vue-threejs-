<script setup lang="ts">
/**
    线条材质:
        1.LineBasicMaterial
        2.LineDashedMaterial 
    注意：设计坐标点都是 x,y,z
*/
import * as Three from 'three';

// 创建渲染器并加入DOM
const render = new Three.WebGLRenderer()
render.setSize(window.innerWidth/2, window.innerHeight/2)
document.body.append(render.domElement)

// 创建透视相机
const cm: any = new Three.PerspectiveCamera(50, window.innerWidth/window.innerHeight, 0.1, 1000)
cm.position.set(0,0,100)
// 相机焦点方向
cm.lookAt( 0, 0, 0 );
// 创建场景
const scene = new Three.Scene()
// 创建线条材质
const mater = new Three.LineBasicMaterial( { color: 0xff0000 } );

// 创建点
const points: any[] = [];
points.push( new Three.Vector3( - 10, 0, 0 ) );
points.push( new Three.Vector3( 0, 10, 0 ) );
points.push( new Three.Vector3( 10, 0, 0 ) );

// 拿着点创建成顶点
const geometry = new Three.BufferGeometry().setFromPoints( points );

// 创建线条
const line = new Three.Line( geometry, mater );
// 添加到场景
scene.add(line)

// 开始渲染
render.render( scene, cm );
</script>

<template>
<h1>线条</h1>
</template>

<style scoped>
header {
  line-height: 1.5;
}


</style>
