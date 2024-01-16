<script setup lang="ts">
import * as THREE from 'three';
// 旧版本引入依赖库，官网有新版本
import Stats from 'three/examples/jsm/libs/stats.module';
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js';
// 引入加载器
// models/gltf/RobotExpressive/RobotExpressive.glb
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { nextTick } from 'vue';

let container, stats, clock, gui, mixer, actions, activeAction, previousAction;
let camera, scene, renderer, model, face;

// 设置面板初始状态对象(非机器人当前状态)
const api = { '当前状态': 'Dance'};

function init() {
    container = document.createElement( 'div' );
    document.body.appendChild( container );
    // 创建相机 45deg 窗口宽/窗口高 近0.25 远100
    camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 0.25, 100 );
    // 移动相机向-5 向上3 向外(前)10
    camera.position.set( -5, 3, 50 );
    // 相机焦点方向（指向Y轴）
	camera.lookAt( 0, 2, 0 );
    // 创建舞台
    scene = new THREE.Scene();
    // 舞台背景色
    scene.background = new THREE.Color( 0xe0e0e0 );
    // 定义舞台中每个舞台的雾 Fog 雾(颜色，近，远)
    scene.fog = new THREE.Fog( 0xe0e0e0, 20, 100 );

    // 跟踪时间
    clock = new THREE.Clock();

    // 灯光 HemisphereLight半球光(天空，地面，暂定光线宽度) 从天空到地面渐变色
    const hemiLight = new THREE.HemisphereLight( 0xffffff, 0x8d8d8d, 3 );
    // 移动灯光位置
    hemiLight.position.set( 0, 20, 0 );
    // 加入舞台
    scene.add( hemiLight );
    // 平行光(颜色、暂定光线宽度)
    const dirLight = new THREE.DirectionalLight( 0xffffff, 3 );
    dirLight.position.set( 0, 20, 10 );
    scene.add( dirLight );

    // 创建物体网格
    /**
        PlaneGeometry：生成平面几何物体(width，height)
        MeshPhongMaterial: 具有镜面高光的光泽表面的材质(options：{color,depthWrite(是否遮挡)...})
    */
    const mesh = new THREE.Mesh( 
        new THREE.PlaneGeometry( 2000, 2000 ), 
        new THREE.MeshPhongMaterial( { color: 0xcbcbcb, depthWrite: false } ) 
        );
    // 沿X轴旋转
    mesh.rotation.x = - Math.PI / 2;
    scene.add( mesh );
    // 坐标格辅助对象 GridHelper(坐标格尺寸, 坐标格细分次数, 中线颜色,坐标格网格线颜色)
    const grid = new THREE.GridHelper( 200, 40, 0x000000, 0x000000 );
    // 透明度
    grid.material.opacity = 0.2;
    grid.material.transparent = true;
    scene.add( grid );

    // 构建gltf loader 
    const loader = new GLTFLoader();
    
    // 加载模型对象(模型文件路径, 回调函数(模型对象))
    loader.load( '/glbs/Parrot.glb', function ( gltf ) {
        
        // gltf模型场景
        model = gltf.scene;
        // 将模型场景加入自定义场景
        scene.add( model );

        createGUI( model, gltf.animations );

    }, undefined, function ( e ) {
        console.error( e );
    } );
    // 创建渲染器 antialias - 是否执行抗锯齿
    renderer = new THREE.WebGLRenderer( { antialias: true } );
    // 设置设备像素比。通常用于避免HiDPI设备上绘图模糊
    renderer.setPixelRatio( window.devicePixelRatio );
    // 窗口大小
    renderer.setSize( window.innerWidth, window.innerHeight );
    // 加入HTMl
    container.appendChild( renderer.domElement );

    window.addEventListener( 'resize', onWindowResize );

    // stats 性能监视器 创建
    stats = new Stats();
    // 加入
    container.appendChild( stats.dom );
}
function animate() {
    const dt = clock.getDelta();

    if ( mixer ) mixer.update( dt );

    requestAnimationFrame( animate );

    renderer.render( scene, camera );

    stats.update();
}
function createGUI(model, animations) {//glft模型场景，glft动作
    console.log('model', model);
    console.log('animations', animations);
    // 动作
    const states = [ 'Idle', 'Walking', 'Running', 'Dance', 'Death', 'Sitting', 'Standing' ];
    // 表达
    const emotes = [ 'Jump', 'Yes', 'No', 'Wave', 'Punch', 'ThumbsUp' ];

    // GUI 方便我们调试 three.js 网页右上角的Controls
    gui = new GUI();

    // 动画混合器
    mixer = new THREE.AnimationMixer( model );

    actions = {};
    // 遍历glft动作 
    for ( let i = 0; i < animations.length; i ++ ) {
        //动作动画切片(还不能被手动调用)
        const clip = animations[ i ];
        // mixer.clipAction 返回传入参数的可以被调用的AnimationAction对象
        const action = mixer.clipAction( clip );
        actions[ clip.name ] = action;

        if ( emotes.indexOf( clip.name ) >= 0 || states.indexOf( clip.name ) >= 4 ) {
            // 不设置clampWhenFinished 暂停动画后 回恢复到开始状态(没有任何动画的状态)
            action.clampWhenFinished = true;
            //让动画不循环播放 Death Sitting Standing
            action.loop = THREE.LoopOnce;
        }

    }    

    // 向调试工具增加菜单头
    const statesFolder = gui.addFolder( '状态' );
    // 增加一个select内容
    const clipCtrl = statesFolder.add( api, '当前状态' ).options( states );

    // 状态发生变化回调
    clipCtrl.onChange( function () {

        fadeToAction( api['当前状态'], 1 );

    } );

    // 未发现有用之处
    statesFolder.open();

    // 增加一个list内容
    const emoteFolder = gui.addFolder( '情感表达' );
    for ( let i = 0; i < emotes.length; i ++ ) {

        createEmoteCallback( emotes[ i ] );

    }    
    // 未发现有用之处
    emoteFolder.open();
    function createEmoteCallback( name ) {
        // 加入面板选项对象
        api[ name ] = function () {
            // 根据传入name修改当前状态
            fadeToAction( name, 0.2 );
            // 监听全部动作完成
            mixer.addEventListener( 'finished', restoreState );

        };
        // 加入面板
        emoteFolder.add( api, name );

    }

    function restoreState() {
        // 移除监听
        mixer.removeEventListener( 'finished', restoreState );
        // 恢复当前动作
        fadeToAction( api['当前状态'], 0.2 );

    }

    // expressions
    console.log('model', model);
    
    face = model.getObjectByName( 'Head_4' );
    console.log('face', face);
    const expressions = Object.keys( face.morphTargetDictionary );
    const expressionFolder = gui.addFolder( 'Expressions' );

    for ( let i = 0; i < expressions.length; i ++ ) {

        expressionFolder.add( face.morphTargetInfluences, i, 0, 1, 0.01 ).name( expressions[ i ] );

    }

    activeAction = actions[ 'Dance' ];
    activeAction.play();

    expressionFolder.open();
}
// 状态发生改变调用的函数
function fadeToAction( name, duration ) {// 修改后的状态，动画时间
    
    // 保存上一个动作
    previousAction = activeAction;
    // 修改机器人动作为传入动作对象
    activeAction = actions[ name ];

    if ( previousAction !== activeAction ) {//修改前后动作不一样
        // 在传入的时间间隔内，逐渐将此动作的权重（weight）由1降至0 可以理解关闭上一个动作
        previousAction.fadeOut( duration );

    }

    activeAction
        .reset() // 重置动作
        .setEffectiveTimeScale( 1 ) // 设置时间比例（timeScale）以及停用所有的变形
        .setEffectiveWeight( 1 ) // 设置权重（weight）以及停止所有淡入淡出
        .fadeIn( duration ) // 在传入的时间间隔内，逐渐将此动作的权重（weight）由0升到1 与fadeOut 相反
        .play();// 播放动画

}

function onWindowResize() {

    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();

    renderer.setSize( window.innerWidth, window.innerHeight );

}


nextTick(() => {
    init();
	animate();
})
</script>

<template>
    <div id="info">
			<!-- <a href="https://threejs.org" target="_blank" rel="noopener">three.js</a> webgl - skinning and morphing<br />
			<p>
				The animation system allows clips to be played individually, looped, or crossfaded with other clips. This example shows a character looping in one of several base animation states, then transitioning smoothly to one-time actions. Facial expressions are controlled independently with morph targets.
			</p>
			Model by
			<a href="https://www.patreon.com/quaternius" target="_blank" rel="noopener">Tomás Laulhé</a>,
			modifications by <a href="https://donmccurdy.com/" target="_blank" rel="noopener">Don McCurdy</a>. CC0.<br /> -->
		</div>
</template>

<style scoped>
    body {
        color: #222;
    }

    a {
        color: #2fa1d6;
    }

    p {
        max-width: 600px;
        margin-left: auto;
        margin-right: auto;
        padding: 0 2em;
    }

</style>: { scene: any; animations: any; }: { isMesh: any; castShadow: boolean; }: any: any: any: any: any: any: { stop: () => void; }: { play: () => void; }: { paused: boolean; }: { paused: boolean; }: any: any: number: any: any: any: any: { action: any; }: { crossFadeTo: (arg0: any, arg1: any, arg2: boolean) => void; }: { time: number; }: any: { enabled: boolean; setEffectiveTimeScale: (arg0: number) => void; setEffectiveWeight: (arg0: any) => void; }: number