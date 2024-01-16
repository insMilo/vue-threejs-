<script setup lang="ts">
import * as THREE from 'three';
import Stats from 'three/examples/jsm/libs/stats.module';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { nextTick, ref, onMounted } from 'vue';
// three/addons/controls/OrbitControls.js
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { RoomEnvironment } from 'three/examples/jsm/environments/RoomEnvironment';
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader';

let mixer;
// 跟踪时间
const clock = new THREE.Clock();
// 定义dom ref
const container = ref<HTMLElement|null>(null)
// 创建性能监测面板
const stats = new Stats()
// 创建渲染器
const renderer = new THREE.WebGLRenderer( {antialias: true} )// 抗锯齿
renderer.setPixelRatio( window.devicePixelRatio );
renderer.setSize( window.innerWidth/2, window.innerHeight/2 );
// 生成环境贴图
const pmremGenerator = new THREE.PMREMGenerator( renderer );
// 创建场景
const scene = new THREE.Scene();
// 设置背景色
scene.background = new THREE.Color( 0xbfe3dd );
// scene.environment 设置所有物理材质的环境贴图
scene.environment = pmremGenerator.fromScene( new RoomEnvironment( renderer ), 0.04 ).texture;// 返回WebGLRendererTarget
// 创建相机
const camera = new THREE.PerspectiveCamera( 40, window.innerWidth / window.innerHeight,00 ); 1, 1
// 调整位置
camera.position.set( 5, 2, 8 );
// （轨道控制器）可以使得相机围绕目标进行轨道运动。
const controls = new OrbitControls( camera, renderer.domElement );
onMounted(() => {
    // 获取DOM
    container.value.appendChild(stats.dom)
    container.value.appendChild(renderer.domElement)
})
</script>
<template>
    <div ref="container"></div>
</template>
<style scoped>

</style>