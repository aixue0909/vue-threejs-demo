<template>
  <div ref="modelDiv" style="width: 500px;height:400px;position:absolute;left:50%;top:50%;transform: translate(-50%,-50%);"></div>
</template>

<script>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
const ThreeBSP = require("tthreebsp")(THREE);
import { Fire } from "three/examples/jsm/objects/Fire";

export default {
  name: "ThreeDemo",
  components: {},
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      controls: null,
      tween: null,
      // door: null,
      //keyboard: new THREEx.KeyboardState(),
      clock: new THREE.Clock(),
      SCREEN_WIDTH: 0,
      SCREEN_HEIGHT: 0,
      VIEW_ANGLE: 75,
      NEAR: 1,
      FAR: 5000,
      materialArrayA: [],
      materialArrayB: [],
      fire: null,
      storey: 10,
      group: new THREE.Group()
    };
  },
  created() {},
  mounted() {
    this.$nextTick(() => {
      this.SCREEN_WIDTH = this.$refs.modelDiv.innerWidth || 500;
      this.SCREEN_HEIGHT = this.$refs.modelDiv.innerHeight || 400;
      this.init();
      this.animate();
    });
  },
  methods: {
    init() {
      // 初始化场景
      this.initScene();
      // 初始化相机
      this.initCamera();
      // 初始化渲染器
      this.initRender();
      // 初始化灯光
      this.initLight();
      // 初始化控制器
      this.initControls();
      // 初始化事件
      this.initEvent();
      // 初始化Object
      this.initObject();
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
      //TWEEN.update()
    },
    initScene() {
      this.scene = new THREE.Scene();
    },
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(
        this.VIEW_ANGLE,
        this.SCREEN_WIDTH / this.SCREEN_HEIGHT,
        this.NEAR,
        this.FAR
      );
      this.camera.position.set(-600, 0, 2000);
      this.camera.lookAt(0, 0, 300);
    },
    initRender() {
      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(this.SCREEN_WIDTH, this.SCREEN_HEIGHT);
      this.$refs.modelDiv.appendChild(this.renderer.domElement);
      this.renderer.setClearColor(0x4682b4, 1.0);
    },
    initEvent() {
      // THREEx.WindowResize(renderer,camera)
      // THREEx.FullScreen.bindKey({charCode: 'm'.charCodeAt(0)})
      document.addEventListener("keydown", this.onKeyDown, false);
    },
    onKeyDown(e) {
      switch (e.keyCode) {
        case 13:
          this.fire.expansion += 0.1;
          if (this.fire.expansion > 1) this.fire.expansion = 0;
          break;
        case 32:
          this.fire.position.set(
            Math.random() * 600,
            Math.random() * 1000,
            310
          );
          break;
        default:
          console.log(e.keyCode);
          break;
      }
    },
    initControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
    },
    initLight() {
      // 位置不同，方向光作用于物体的面也不同，看到的物体各个面的颜色也不同
      // A start, 第二个参数是光源强度
      let directionalLight = new THREE.DirectionalLight(0x707070, 1); //模拟远处类似太阳的光源
      directionalLight.position.set(0, 100, 0).normalize();
      this.scene.add(directionalLight);
      //A end
      let ambient = new THREE.AmbientLight(0x333333, 1); //AmbientLight,影响整个场景的光源
      ambient.position.set(0, 0, 0);
      this.scene.add(ambient);
    },
    initObject() {
      this.createWallMaterial(); // 创建纹理
      this.createFloor(); // 创建地板
      this.createLayout(); // 创建布局
    },
    createWallMaterial() {
      this.materialArrayA.push(
        new THREE.MeshPhongMaterial({ color: 0xafc0ca })
      ); //前  0xafc0ca :灰色
      this.materialArrayA.push(
        new THREE.MeshPhongMaterial({ color: 0xafc0ca })
      ); //后
      this.materialArrayA.push(
        new THREE.MeshPhongMaterial({ color: 0xd6e4ec })
      ); //上  0xd6e4ec： 偏白色
      this.materialArrayA.push(
        new THREE.MeshPhongMaterial({ color: 0xd6e4ec })
      ); //下
      this.materialArrayA.push(
        new THREE.MeshPhongMaterial({ color: 0xafc0ca })
      ); //左    0xafc0ca :灰色
      this.materialArrayA.push(
        new THREE.MeshPhongMaterial({ color: 0xafc0ca })
      ); //右

      this.materialArrayB.push(
        new THREE.MeshPhongMaterial({ color: 0xafc0ca })
      ); //前  0xafc0ca :灰色
      this.materialArrayB.push(
        new THREE.MeshPhongMaterial({ color: 0x9cb2d1 })
      ); //后  0x9cb2d1：淡紫
      this.materialArrayB.push(
        new THREE.MeshPhongMaterial({ color: 0xd6e4ec })
      ); //上  0xd6e4ec： 偏白色
      this.materialArrayB.push(
        new THREE.MeshPhongMaterial({ color: 0xd6e4ec })
      ); //下
      this.materialArrayB.push(
        new THREE.MeshPhongMaterial({ color: 0xafc0ca })
      ); //左   0xafc0ca :灰色
      this.materialArrayB.push(
        new THREE.MeshPhongMaterial({ color: 0xafc0ca })
      ); //右
    },
    createFloor() {
      let loader = new THREE.TextureLoader();
      let texture1 = loader.load("images/floor.jpg");
      //console.log(texture)
      texture1.wrapS = texture1.wrapT = THREE.RepeatWrapping;
      texture1.repeat.set(2, 2);
      let floorGeometry1 = new THREE.BoxGeometry(2000, 1000, 0.1);
      let floorMaterial1 = new THREE.MeshBasicMaterial({
        map: texture1,
        side: THREE.DoubleSide
      });
      let floor1 = new THREE.Mesh(floorGeometry1, floorMaterial1);
      floor1.position.y = -0.05 - this.storey * 100 - 100;
      floor1.rotation.x = Math.PI / 2;
      this.group.add(floor1);

      let texture2 = loader.load("images/floor1.jpg");

      texture2.wrapS = texture2.wrapT = THREE.RepeatWrapping;
      texture2.repeat.set(5, 5);
      let floorGeometry2 = new THREE.BoxGeometry(1200, 600, 1);
      let floorMaterial2 = new THREE.MeshBasicMaterial({
        map: texture2,
        side: THREE.DoubleSide
      });
      let floor2 = new THREE.Mesh(floorGeometry2, floorMaterial2);
      floor2.position.y = -this.storey * 100 - 100;
      floor2.rotation.x = Math.PI / 2;
      this.group.add(floor2);
    },
    createLayout() {
      // 创建楼层
      for (let i = 0; i < this.storey; i++) {
        let obj = this.createStorey(i);
        this.group.add(obj);
      }
      // 创建隔层
      let gc6 = this.returnWallObject(
        1240,
        20,
        640,
        0,
        this.materialArrayA,
        0,
        90,
        0
      );
      let gc8 = this.returnWallObject(
        1240,
        20,
        640,
        0,
        this.materialArrayA,
        0,
        490,
        0
      );
      this.group.add(gc6);
      this.group.add(gc8);
      // 封顶
      let topWall = this.returnWallObject(
        1200,
        10,
        600,
        0,
        this.materialArrayB,
        0,
        100 * this.storey - 150,
        0
      );
      this.group.add(topWall);
      // 按门
      let loader = new THREE.TextureLoader();
      let texture1 = loader.load("images/door_right.png");

      let doorgeometry = new THREE.BoxGeometry(100, 200, 2);
      let doormaterial1 = new THREE.MeshPhongMaterial({
        map: texture1,
        color: 0xffffff,
        opacity: 1,
        transparent: true
      });
      let door1 = new THREE.Mesh(doorgeometry, doormaterial1);
      door1.position.set(50, -1000, 300);
      this.group.add(door1);

      let texture2 = loader.load("images/door_left.png");

      // let doorgeometry = new THREE.BoxGeometry(100,200,2)
      let doormaterial2 = new THREE.MeshPhongMaterial({
        map: texture2,
        color: 0xffffff,
        opacity: 1,
        transparent: true
      });
      let door2 = new THREE.Mesh(doorgeometry, doormaterial2);
      door2.position.set(-50, -1000, 300);
      this.group.add(door2);

      // 火焰
      let plane = new THREE.PlaneBufferGeometry(200, 300);
      this.fire = new Fire(plane, {
        textureWidth: 512,
        textureHeight: 512,
        debug: false
      });
      this.fire.rotation.y = 0;
      this.fire.position.set(0, 200, 310);
      this.fire.expansion = 0.4;
      this.fire.windVector.y = 0;
      this.fire.windVector.x = 0;
      this.fire.scale.x = 0.8;
      this.fire.addSource(0.5, 0.1, 0.1, 1.0, 3.0, 5.0);
      this.group.add(this.fire);
      this.scene.add(this.group);
    },
    createStorey(index) {
      // 创建围墙
      let storeyGroup = new THREE.Group();
      let leftWall = this.returnWallObject(
        10,
        200,
        600,
        0,
        this.materialArrayB,
        -600,
        -100 * this.storey + index * 200,
        0
      );
      let rightWall = this.returnWallObject(
        10,
        200,
        600,
        1,
        this.materialArrayB,
        600,
        -100 * this.storey + index * 200,
        0
      );
      let backWall = this.returnWallObject(
        1200,
        200,
        10,
        0,
        this.materialArrayB,
        0,
        -100 * this.storey + index * 200,
        -300
      );
      let frontWall = this.returnWallObject(
        1200,
        200,
        10,
        0,
        this.materialArrayB,
        0,
        -100 * this.storey + index * 200,
        300
      );
      let glassDoor, glassWindow1, glassWindow4;
      if (index == 0) {
        glassDoor = this.returnWallObject(
          200,
          200,
          10,
          0,
          this.materialArrayB,
          0,
          -100 * this.storey + index * 200,
          300
        );
      } else {
        glassWindow1 = this.returnWallObject(
          60,
          60,
          10,
          0,
          this.materialArrayB,
          100,
          -100 * this.storey + index * 200,
          300
        );
        glassWindow4 = this.returnWallObject(
          60,
          60,
          10,
          0,
          this.materialArrayB,
          -100,
          -100 * this.storey + index * 200,
          300
        );
      }
      let glassWindow2 = this.returnWallObject(
        60,
        60,
        10,
        0,
        this.materialArrayB,
        300,
        -100 * this.storey + index * 200,
        300
      );
      let glassWindow3 = this.returnWallObject(
        60,
        60,
        10,
        0,
        this.materialArrayB,
        500,
        -100 * this.storey + index * 200,
        300
      );
      let glassWindow5 = this.returnWallObject(
        60,
        60,
        10,
        0,
        this.materialArrayB,
        -300,
        -100 * this.storey + index * 200,
        300
      );
      let glassWindow6 = this.returnWallObject(
        60,
        60,
        10,
        0,
        this.materialArrayB,
        -500,
        -100 * this.storey + index * 200,
        300
      );
      let resultWall;
      resultWall = this.returnResultBsp(frontWall, glassWindow2);
      if (glassWindow1 && glassWindow4) {
        resultWall = this.returnResultBsp(resultWall, glassWindow1);
        resultWall = this.returnResultBsp(resultWall, glassWindow4);
      }
      if (glassDoor) resultWall = this.returnResultBsp(resultWall, glassDoor);
      resultWall = this.returnResultBsp(resultWall, glassWindow3);
      resultWall = this.returnResultBsp(resultWall, glassWindow5);
      resultWall = this.returnResultBsp(resultWall, glassWindow6);

      storeyGroup.add(resultWall);
      storeyGroup.add(leftWall);
      storeyGroup.add(rightWall);
      storeyGroup.add(backWall);
      // 装玻璃
      let glassGeo = new THREE.BoxGeometry(60, 60, 2);
      let glassMaterial = new THREE.MeshBasicMaterial({
        color: 0xecf1f3,
        transparent: true,
        opacity: 0.5
      });
      let glass = new THREE.Mesh(glassGeo, glassMaterial);
      let glass1 = glass.clone();
      let glass2 = glass.clone();
      let glass3 = glass.clone();
      let glass4 = glass.clone();
      let glass5 = glass.clone();
      glass.position.set(100, -100 * this.storey + index * 200, 300);
      glass3.position.set(-100, -100 * this.storey + index * 200, 300);
      glass1.position.set(300, -100 * this.storey + index * 200, 300);
      glass2.position.set(500, -100 * this.storey + index * 200, 300);
      glass4.position.set(-300, -100 * this.storey + index * 200, 300);
      glass5.position.set(-500, -100 * this.storey + index * 200, 300);
      let glassGroup = new THREE.Group();

      if (index != 0) {
        glassGroup.add(glass);
        glassGroup.add(glass3);
      }
      glassGroup.add(glass1);
      glassGroup.add(glass2);
      glassGroup.add(glass4);
      glassGroup.add(glass5);
      storeyGroup.add(glassGroup);

      return storeyGroup;
      //}
      // let result =  group.toJSON()
      // let loader = new THREE.ObjectLoader()
      // let obj = loader.parse(result)
      // scene.add(group)
      // let fire = new THREE.Fire(glass)
    },

    // 创建墙
    createCubeWall(width, height, depth, angle, material, x, y, z) {
      let cubeGeometry = new THREE.BoxGeometry(width, height, depth);
      let cube = new THREE.Mesh(cubeGeometry, material);
      cube.position.x = x;
      cube.position.y = y;
      cube.position.z = z;
      cube.rotation.y += angle * Math.PI; // 正逆时针转 ﹣顺时针转
      this.group.add(cube);
    },
    returnWallObject(width, height, depth, angle, material, x, y, z) {
      let cubeGeometry = new THREE.BoxGeometry(width, height, depth);
      let cube = new THREE.Mesh(cubeGeometry, material);
      cube.position.x = x;
      cube.position.y = y;
      cube.position.z = z;
      cube.rotation.y += angle * Math.PI; // 正逆时针转 ﹣顺时针转
      return cube;
    },
    returnResultBsp(bsp, lessBsp, mat) {
      // bsp要处理的几何体、lessBsp要抠去的几何体、材料标识
      let material;
      switch (
        mat // 先判断需要什么材质
      ) {
        case 1:
          material = new THREE.MeshPhongMaterial({
            color: 0x9cb2d1,
            specular: 0x9cb2d1,
            shininess: 30,
            transparent: true,
            opacity: 1
          });
          break;
        case 2:
          material = new THREE.MeshPhongMaterial({
            color: 0xafc0ca,
            specular: 0xafc0ca,
            shininess: 30,
            transparent: true,
            opacity: 1
          });
          break;
        default:
      }
      let bsp1 = new ThreeBSP(bsp);
      let bsp2 = new ThreeBSP(lessBsp);
      let resultBsp = bsp1.subtract(bsp2);
      let result = resultBsp.toMesh(material); // 转换成mesh

      result.material.flatshading = THREE.FlatShading; // 平面底纹
      result.geometry.computeFaceNormals(); //重新计算几何体侧面法向量
      result.geometry.computeVertexNormals();
      result.material.needsUpdate = true; //更新纹理
      result.geometry.buffersNeedUpdate = true;
      result.geometry.uvsNeedUpdate = true;
      return result;
    }
  }
};
</script>

<style>
body {
  margin: 0;
  overflow: hidden;
}
</style>
