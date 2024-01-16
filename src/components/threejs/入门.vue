<script setup lang="ts">
import * as THREE from 'three';
console.log('THREE', THREE);
/**
 *  Three 笔记
    1. 基本要素 场景(舞台)（scene）渲染器（renderer) 相机(camera)

    camera：
      1.PerspectiveCamera(透视相机) 
        - 参数
          (1) 视野角度（FOV）: 透过显示器看到的范围 (单位是角度)
          (2) 长宽比（aspect ratio）例如16/9 4/3 width/height
          (3) 近截面（near）
          (4) 远截面（far）
            - 注意
              a. 当内容比远截面远或者比近截面近时是不会被渲染到舞台上
    
    renderer：
      1.WebGLRenderer(最高级渲染器，对老版本浏览器兼容性不够)
      

      方法：renderer.setSize(设置窗口width，设置窗口height，保持应用程序的尺寸，但是以较低的分辨率来渲染（false）)
        - 注意：对于性能比较敏感的应用程序来说，需要传入的值小一点，可以以浏览器窗口一半进行渲染
                设置canvas 宽度为100%，设置宽高为原来的一半 那么渲染比例为原来1/4

    模型
      1.BoxGeometry 立方体


    材质
      1.MeshBasicMaterial 网格材质(可以设置颜色)


    网格
      1.Mesh(网格) 可以直接展示到场景之中的介质
 */
// init
// 创建场景(舞台)
const scene = new THREE.Scene();
// 创建相机
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
// 创建渲染器
const renderer = new THREE.WebGLRenderer();
// 通过setSize 设置渲染器窗口大小
renderer.setSize( window.innerWidth/2, window.innerHeight/2 );

// 创建立方体
const geometry = new THREE.BoxGeometry( 1, 1, 1 );// 是长宽高
const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );// 设置材质颜色 0x 16进制
const cube = new THREE.Mesh( geometry, material );
// 调用scene.add 默认网格坐标为(0,0,0) 所以通过设置camera.position.z 使相机向外移动一些（可以控制图像看到的大小）
scene.add( cube );
camera.position.z = 3;

// 加入dom元素,但是内容不会真正的被渲染
document.body.appendChild( renderer.domElement );

// 真正渲染需要通过 渲染循环”（render loop）或者“动画循环”（animate loop）实现渲染
function animate() {
  // requestAnimationFrame 控制渲染时间,每60s渲染一次，同样可以使用setInterval 实现，但是requestAnimationFrame较好
  // 会在切换标签页面自动停止渲染
	requestAnimationFrame( animate );
  /**
    使图像动起来
  */
  cube.rotation.x += 0.01; // 上下翻滚
  cube.rotation.y += 0.01;// 左右翻滚
	renderer.render( scene, camera );
}
animate();
</script>

<template>
<h1>入门</h1>
</template>

<style scoped>
header {
  line-height: 1.5;
}


</style>
