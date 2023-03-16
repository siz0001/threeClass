<script setup>
import { ref, onMounted, onBeforeUnmount, watch } from 'vue'
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { PointerLockControls } from "three/examples/jsm/controls/PointerLockControls"
import { OrbitControlsGizmo } from '@/plugins/OrbitControlsGizmo'
import { TransformControls } from 'three/examples/jsm/controls/TransformControls.js';
import { InteractionManager } from 'three.interactive';
import { ShadowMapViewer } from "three/examples/jsm/utils/ShadowMapViewer"
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader"
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader"
import { FBXLoader } from "three/examples/jsm/loaders/FBXLoader"
// aa1

const canvas = ref();
const cameraValue = ref();
const rendererValue = ref()
const interactionManager = ref()
const controls = ref({ isLocked: false })
const winResize = ref()
const popup = ref(false)
const popup2 = ref(false)
const popup3 = ref(false)
const popup4 = ref(false)
const popup5 = ref(false)
const popup6 = ref(false)
const popupData = ref()
const popup3Data = ref()
const clickcheck = ref()
const dialogXY = ref([0, 0])

const shadowX = ref(400 * Math.cos(1))
const shadowY = ref(400 * Math.sin(1))
const shadowZ = ref(0)
const lightPosition = ref(0)
const spotLight = ref()
const mousePosX = ref()
const mousePosY = ref()
const controlsGizmo = ref()
const selectedMesh = ref([])
const selectedMeshColor = ref()
const selectedMeshHoverColor = ref()
const selectedMeshOpacity = ref()
const selectedMeshShininess = ref()
const lightsList = ref([])
const lightType = ref([0, 0, 0, 0, 0])
const ambientLightsList = ref([])
const directionalLightsList = ref([])
const hemisphereLightsList = ref([])
const pointLightsList = ref([])
const spotLightsList = ref([])
const selectedLight = ref({ intensity: 0 })
const selectedLightNo = ref()
const selectedLightColor = ref()
const controlValue = ref()
const hoverAction = ref(true)
const gltfModel = ref()
const gltfModel2 = ref()
const gltfModel3 = ref()
const planeMesh = ref()
const boxMesh = ref()
const textureURL = ref('/texture.png')

const windowResize = (renderer, camera, dimension) => {
  dimension = dimension || function () { return { width: window.innerWidth, height: window.innerHeight } }
  var callback = function () {
    var rendererSize = dimension();
    renderer.setSize(rendererSize.width, rendererSize.height)
    camera.aspect = rendererSize.width / rendererSize.height
    camera.updateProjectionMatrix()
  }
  window.addEventListener('resize', callback, false)
  return {
    trigger: function () {
      callback()
    },
    destroy: function () {
      window.removeEventListener('resize', callback)
    }
  }
}

const createModel = (scene) => {
  const loader = new GLTFLoader();
  loader.load('/01.gltf', (gltf) => {
    gltf.scene.traverse(function (node) {
      if (node.isMesh) { node.castShadow = true; node.receiveShadow = true; }
    });

    scene.add(gltf.scene);
    gltfModel.value = gltf.scene;
  })
}

const createModel2 = (scene) => {
  const loader = new GLTFLoader();
  loader.load('/botcherShop.glb', (gltf) => {
    gltf.scene.traverse(function (node) {
      if (node.isMesh) {
        node.castShadow = true;
        node.receiveShadow = true;
        node.scale.x = 3;
        node.scale.y = 3;
        node.scale.z = 3;
        node.position.x = -15;
      }
    });
    scene.add(gltf.scene);
    gltfModel2.value = gltf.scene;
  })
}

const createModel3 = (scene) => {
  const loader = new FBXLoader();
  loader.load('/FBX/FuelTruck/FuelTruck.fbx', (fbx) => {
    fbx.traverse(function (node) {
      if (node.isMesh) {
        node.castShadow = true;
        node.receiveShadow = true;
      }    
    });
    scene.add(fbx);
    gltfModel3.value = fbx;
    gltfModel3.value.position.x = 10;
    gltfModel3.value.scale.x = 3;
    gltfModel3.value.scale.y = 3;
    gltfModel3.value.scale.z = 3;
    console.log(gltfModel3.value.scale)
  })

}



const createMeshPlane = (scene) => {
  var groundTexture = new THREE.TextureLoader().load(textureURL.value);
  const plane = new THREE.Mesh(new THREE.PlaneGeometry(100, 100), new THREE.MeshPhongMaterial({
    map: groundTexture,
    // color: 0xffffff,
    side: THREE.DoubleSide,
    shininess: 0
  }));

  plane.rotateX(3 * Math.PI / 2)
  plane.castShadow = false;
  plane.receiveShadow = true;
  scene.add(plane);
  planeMesh.value = plane
}

const createMeshBox = (scene) => {
  const box = new THREE.Mesh(new THREE.BoxGeometry(10, 10, 10), new THREE.MeshPhongMaterial({
    color: 0xffffff,
    // side: THREE.DoubleSide,
  }));
  box.position.x = 20
  box.position.y = 20
  box.position.z = 10
  box.rotateX(3 * Math.PI / 2)
  box.castShadow = true;
  box.receiveShadow = true;
  scene.add(box);
  boxMesh.value = box
}

const createScene = () => {
  const scene = new THREE.Scene();
  // 배경색 설정하고 싶을경우
  // scene.background = new THREE.Color("#FFFFFF")
  // scene.background = new THREE.Color().setHSL(0.6, 0, 1);
  scene.fog = new THREE.Fog(scene.background, 1, 5000);
  return scene
}
const scene = createScene();


const createCamera = () => {
  let innerWidth = canvas.value.offsetWidth;
  let innerHeight = canvas.value.offsetHeight;
  const camera = new THREE.PerspectiveCamera(75, innerWidth / innerHeight, 0.1, 1000);
  camera.position.z = 100;
  camera.position.y = 50;

  return { camera, innerWidth, innerHeight };
}
const createLight = (scene, camera, renderer) => {
  scene.add(new THREE.AmbientLight(0x404040));
  const size = window.innerWidth * 0.15;

  spotLight.value = new THREE.SpotLight(0xffffff);
  spotLight.value.angle = Math.PI / 5;
  spotLight.value.penumbra = 0.3;
  spotLight.value.position.set(350, 350, 350);
  spotLight.value.castShadow = true;
  // spotLight.value.shadow.radius = 100;
  spotLight.value.shadow.camera.near = 8;
  spotLight.value.shadow.camera.far = 550;
  spotLight.value.shadow.mapSize.width = 6000;
  spotLight.value.shadow.mapSize.height = 6000;

  scene.add(spotLight.value);
  scene.add(spotLight.value.target);

  const spotLightShadowMapViewer = new ShadowMapViewer(spotLight);
  spotLightShadowMapViewer.size.set(size, size);
  spotLightShadowMapViewer.position.set(size + 200, 100);

  for (var i = 0; i < 5; i += 1) {
    // lightsList.value[i] = new THREE.AmbientLight(0x404040); // soft white light
    lightsList.value[i] = new THREE.PointLight(0xff0000, 1, 100);
    lightsList.value[i].castShadow = true;
    lightsList.value[i].visible = false;
    lightsList.value[i].position.set(0, 50, 0);
    scene.add(lightsList.value[i]);
  }
  for (var i = 0; i < 5; i += 1) {

    spotLightsList.value[i] = new THREE.AmbientLight(0x404040);
    spotLightsList.value[i].castShadow = true;
    spotLightsList.value[i].visible = false;
    spotLightsList.value[i].position.set(0, 50, 0);
    scene.add(spotLightsList.value[i]);

  }
  for (var i = 0; i < 5; i += 1) {
    ambientLightsList.value[i] = new THREE.AmbientLight(0x404040);
    ambientLightsList.value[i].castShadow = true;
    ambientLightsList.value[i].visible = false;
    ambientLightsList.value[i].position.set(0, 50, 0);
    scene.add(ambientLightsList.value[i]);

  }
  for (var i = 0; i < 5; i += 1) {
    directionalLightsList.value[i] = new THREE.DirectionalLight(0xffffff, 0.5)
    directionalLightsList.value[i].castShadow = true;
    directionalLightsList.value[i].visible = false;
    directionalLightsList.value[i].position.set(0, 50, 0);
    scene.add(directionalLightsList.value[i]);

  }
  for (var i = 0; i < 5; i += 1) {
    hemisphereLightsList.value[i] = new THREE.HemisphereLight(0xffffbb, 0x080820, 1);
    hemisphereLightsList.value[i].castShadow = true;
    hemisphereLightsList.value[i].visible = false;
    hemisphereLightsList.value[i].position.set(0, 50, 0);
    scene.add(hemisphereLightsList.value[i]);

  }
  for (var i = 0; i < 5; i += 1) {

    pointLightsList.value[i] = new THREE.PointLight(0xff0000, 1, 100);
    pointLightsList.value[i].castShadow = true;
    pointLightsList.value[i].visible = false;
    pointLightsList.value[i].position.set(0, 50, 0);
    scene.add(pointLightsList.value[i]);

  }



  const control = new TransformControls(camera, renderer.domElement);
  control.addEventListener('dragging-changed', (event) => {
    controls.value.enabled = !event.value;
  });
  control.attach(lightsList.value[0]);
  control.visible = false
  scene.add(control);
  controlValue.value = control

  /* 
    const dirLight = new THREE.DirectionalLight(0xffffff, 1);
    dirLight.position.set(0, 10, 0);
    dirLight.castShadow = true;
    dirLight.shadow.camera.near = 1;
    dirLight.shadow.camera.far = 10;
    dirLight.shadow.camera.right = 15;
    dirLight.shadow.camera.left = - 15;
    dirLight.shadow.camera.top = 15;
    dirLight.shadow.camera.bottom = - 15;
    dirLight.shadow.mapSize.width = 1024;
    dirLight.shadow.mapSize.height = 1024;
    scene.add(dirLight);
   
    const dirLightShadowMapViewer = new ShadowMapViewer(dirLight);
  
      dirLightShadowMapViewer.position.x = 10;
      dirLightShadowMapViewer.position.y = 10;
      dirLightShadowMapViewer.size.width = size;
      dirLightShadowMapViewer.size.height = size;
   */
}

const createHdr = (scene) => {
  const loader = new RGBELoader();
  loader.load('/puresky_2k.hdr', function (texture) {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.background = texture;
    scene.environment = texture;
  });

}

const createRenderer = (camera, innerWidth, innerHeight) => {
  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.PCFSoftShadowMap;
  renderer.setSize(innerWidth, innerHeight);
  canvas.value.appendChild(renderer.domElement)
  winResize.value = windowResize(renderer, camera)
  renderer.localClippingEnabled = true
  return renderer
}

const handleInit = () => {
  const { camera, innerWidth, innerHeight } = createCamera();
  const renderer = createRenderer(camera, innerWidth, innerHeight)
  interactionManager.value = new InteractionManager(
    renderer,
    camera,
    renderer.domElement
  );

  // camera.lookAt(0, 0, geoJsonStore.bB005Data.elevation+beforeDepth.value)
  controls.value = new OrbitControls(camera, renderer.domElement)
  controls.value.maxPolarAngle = Math.PI / 2;
  controls.value.target = new THREE.Vector3(0, 0, 0)
  controlsGizmo.value = new OrbitControlsGizmo(controls.value, { size: 120, padding: 8 });
  canvas.value.appendChild(controlsGizmo.value.domElement);

  createLight(scene, camera, renderer)
  const spotLightHelper = new THREE.SpotLightHelper(spotLight.value);
  scene.add(spotLightHelper);

  const axesHelper = new THREE.AxesHelper(500);
  scene.add(axesHelper);

  createMeshPlane(scene)
  createModel(scene)
  createModel2(scene)
  createModel3(scene)
  createMeshBox(scene)
  createHdr(scene, renderer);

  function animate() {


    requestAnimationFrame(animate);
    interactionManager.value.update();
    // controls.value.update();
    renderer.render(scene, camera);
    rendererValue.value = renderer
    cameraValue.value = camera
    spotLight.value.position.set(shadowX.value, shadowY.value, shadowZ.value)
    spotLight.value.target.position.set(0, 0, 0)
    spotLightHelper.update();
  }
  animate();
}

const outsideAction = () => {
  if (popup.value && clickcheck.value) {
  } else if (popup.value && !clickcheck.value) {
    popup.value = false
  }
}
const meshButtonAction = (item, value) => {
  if (popup6.value) {
    popup4.value = false
    popup2.value = false
    popup5.value = false
    popup6.value = false

  } else {
    popup4.value = false
    popup2.value = false
    popup5.value = false
    popup6.value = !popup6.value
    selectedMesh.value = item

    selectedMesh.value.forEach(item => {
      if (item[0]) {
        item.forEach(innerItem => {
          console.log(innerItem.material.color.getHexString())
          selectedMeshColor.value = innerItem.material.color.getHexString()
          selectedMeshOpacity.value = innerItem.material.opacity
          selectedMeshShininess.value = innerItem.material.shininess
        })
      } else {
        console.log(selectedMesh.value)
        console.log(item.material.color.getHex())
        selectedMeshColor.value = item.material.color.getHexString()
        selectedMeshOpacity.value = item.material.opacity
        selectedMeshShininess.value = item.material.shininess
      }
    })

  }
  console.log(value)
  if (0 < value) {
    hoverAction.value = value

  } else {
    hoverAction.value = 0

  }
}
const changeMeshVisible = (value) => {

  value.visible = !value.visible

}
const changeMainLightPosition = () => {
  shadowX.value = 400 * Math.cos(lightPosition.value)
  shadowY.value = 400 * Math.sin(lightPosition.value)
}

const lightButtonAction = (i) => {
  if (popup5.value) {
    popup4.value = false
    popup2.value = false
    popup5.value = false
    popup6.value = false
    controlValue.value.visible = false
    controlValue.value.attach(spotLight.value);
  } else {
    popup4.value = false
    popup2.value = false
    popup6.value = false
    popup5.value = !popup5.value

    selectedLightNo.value = i

  }
}



const changeLightColor = () => {
  switch (lightType.value[selectedLightNo.value]) {
    case 1:
      pointLightsList.value[selectedLightNo.value].color.setHex(`0x${selectedLightColor.value.substr(1, 6)} `)
      break;
    case 2:
      spotLightsList.value[selectedLightNo.value].color.setHex(`0x${selectedLightColor.value.substr(1, 6)} `)
      break;
    case 33:
      ambientLightsList.value[selectedLightNo.value].color.setHex(`0x${selectedLightColor.value.substr(1, 6)} `)
      break;
    case 4:
      hemisphereLightsList.value[selectedLightNo.value].color.setHex(`0x${selectedLightColor.value.substr(1, 6)} `)
      break;
    case 5:
      directionalLightsList.value[selectedLightNo.value].color.setHex(`0x${selectedLightColor.value.substr(1, 6)} `)
      break;
    default:
      break;
  }
  console.log(selectedLight.value)
  lightsList.value[selectedLightNo.value].color.setHex(`0x${selectedLightColor.value.substr(1, 6)} `)
}

const changeMeshColor = () => {

  selectedMesh.value.forEach(item => {
    if (item[0]) {
      item.forEach(innerItem => {
        console.log(innerItem.material.color)
        innerItem.material.color.setHex(`0x${selectedMeshColor.value.substr(1, 6)}`)

      })
    } else {
      console.log(selectedMesh.value)
      item.material.color.setHex(`0x${selectedMeshColor.value.substr(1, 6)}`)
    }
  })

}


const changeLightType = () => {

  switch (lightType.value[selectedLightNo.value]) {
    case 0:
      pointLightsList.value[selectedLightNo.value].visible = false
      spotLightsList.value[selectedLightNo.value].visible = false
      ambientLightsList.value[selectedLightNo.value].visible = false
      hemisphereLightsList.value[selectedLightNo.value].visible = false
      directionalLightsList.value[selectedLightNo.value].visible = false
      controlValue.value.visible = false
      selectedLight.value.intensity = 0
      break;

    case 1:
      pointLightsList.value[selectedLightNo.value].visible = true
      spotLightsList.value[selectedLightNo.value].visible = false
      ambientLightsList.value[selectedLightNo.value].visible = false
      hemisphereLightsList.value[selectedLightNo.value].visible = false
      directionalLightsList.value[selectedLightNo.value].visible = false
      controlValue.value.attach(pointLightsList.value[selectedLightNo.value])
      controlValue.value.visible = true
      selectedLight.value = pointLightsList.value[selectedLightNo.value]
      break;
    case 2:
      pointLightsList.value[selectedLightNo.value].visible = false
      spotLightsList.value[selectedLightNo.value].visible = true
      ambientLightsList.value[selectedLightNo.value].visible = false
      hemisphereLightsList.value[selectedLightNo.value].visible = false
      directionalLightsList.value[selectedLightNo.value].visible = false
      controlValue.value.attach(spotLightsList.value[selectedLightNo.value])
      controlValue.value.visible = true
      selectedLight.value = spotLightsList.value[selectedLightNo.value]
      break;
    case 3:
      pointLightsList.value[selectedLightNo.value].visible = false
      spotLightsList.value[selectedLightNo.value].visible = false
      ambientLightsList.value[selectedLightNo.value].visible = true
      hemisphereLightsList.value[selectedLightNo.value].visible = false
      directionalLightsList.value[selectedLightNo.value].visible = false
      controlValue.value.attach(ambientLightsList.value[selectedLightNo.value])
      controlValue.value.visible = true
      selectedLight.value = ambientLightsList.value[selectedLightNo.value]
      break;
    case 4:
      pointLightsList.value[selectedLightNo.value].visible = false
      spotLightsList.value[selectedLightNo.value].visible = false
      ambientLightsList.value[selectedLightNo.value].visible = false
      hemisphereLightsList.value[selectedLightNo.value].visible = true
      directionalLightsList.value[selectedLightNo.value].visible = false
      controlValue.value.attach(hemisphereLightsList.value[selectedLightNo.value])
      controlValue.value.visible = true
      selectedLight.value = hemisphereLightsList.value[selectedLightNo.value]
      break;
    case 5:
      pointLightsList.value[selectedLightNo.value].visible = false
      spotLightsList.value[selectedLightNo.value].visible = false
      ambientLightsList.value[selectedLightNo.value].visible = false
      hemisphereLightsList.value[selectedLightNo.value].visible = false
      directionalLightsList.value[selectedLightNo.value].visible = true
      controlValue.value.attach(directionalLightsList.value[selectedLightNo.value])
      controlValue.value.visible = true
      selectedLight.value = directionalLightsList.value[selectedLightNo.value]
      break;


    default:
      break;
  }


}
watch(selectedMeshOpacity, (newValue, oldValue) => {
  selectedMesh.value.forEach(item => {
    if (item[0]) {
      item.forEach(innerItem => {
        console.log(innerItem.material.opacity)
        innerItem.material.opacity = newValue
      })
    } else {
      console.log(selectedMesh.value)
      item.material.opacity = newValue
    }
  })
})

watch(selectedMeshShininess, (newValue, oldValue) => {
  selectedMesh.value.forEach(item => {
    if (item[0]) {
      item.forEach(innerItem => {
        console.log(innerItem.material.shininess)
        innerItem.material.shininess = newValue
      })
    } else {
      console.log(selectedMesh.value)
      item.material.shininess = newValue
    }
  })
})


onMounted(async () => {
  document.addEventListener("mousemove", (event) => {
    mousePosX.value = event.clientX;
    mousePosY.value = event.clientY;
  });


  handleInit();
})
onBeforeUnmount(() => {
  // new THREE.dispose()
})
</script>

<template>
  <v-app>
    <v-main>
      <div class="three3d" ref="canvas">
      </div>
      <!-- bb1 -->
      <v-navigation-drawer permanent location="right" color="#FFFFFF90" width="400">

        <!-- 핵심 -->
        <v-card flat class="ma-4 pa-2" color="#FFFFFF90">

          <v-row>
            <v-col class="d-flex justify-space-between align-center">
              <v-btn block flat color="#5338EB" class="text-white" @click="changeMeshVisible(boxMesh)">
                박스 메쉬
              </v-btn>
            </v-col>
            <v-col class="d-flex justify-space-between align-center">
              <v-btn icon @click="meshButtonAction(boxMesh)" variant="text" rounded="40" color="#5338EB" class="ma-0"
                size="small">
                <v-icon size="32">mdi-cube</v-icon>
              </v-btn>
            </v-col>

          </v-row>

        </v-card>
        <!-- 보조 -->
        <v-card flat class="ma-4 pa-2" color="#FFFFFF90">
          <v-card-text class="pa-0">
            보조컨트롤 기능
          </v-card-text>
          <v-divider class="my-2"></v-divider>
          <v-row>

            <v-col>
              <v-btn block flat color="#5338EB" @click="
                popup4 = false;
              popup5 = false;
              popup2 = !popup2" class="text-white" style="font-weight:bold; font-size:16px">
                빛 방향
              </v-btn>
            </v-col>
          </v-row>
        </v-card>
        <!-- 광원 -->
        <v-card flat class="ma-4 pa-2" color="#FFFFFF90">
          <v-card-text class="pa-0">
            광원 컨트롤 기능
          </v-card-text>
          <v-divider class="my-2"></v-divider>
          <v-row class="pa-2">
            <v-col v-for="i in 5" :key="i" class="pa-1">
              <v-btn block flat :variant="lightType[i] > 0 ? 'flat' : 'outlined'"
                :class="lightType[i] > 0 ? 'text-white' : ''" color="#5338EB" @click="lightButtonAction(i)">{{ i
                }}</v-btn>
            </v-col>
          </v-row>
        </v-card>

      </v-navigation-drawer>
      <!-- bb2 -->
      <v-card v-click-outside="outsideAction()" v-if="popup"
        :style="'position:absolute; top:' + dialogXY[0] + 'px; left:' + dialogXY[1] + 'px'" elevation="10">

        <v-table density="compact">
          <tbody>
            <thead class="">
              <th class="py-2 pl-4">
                건축물 정보
              </th>
            </thead>
            <tr>
              <td>건폐율</td>
              <td>{{ popupData.cob }}%</td>
            </tr>
            <tr color="grey">
              <td>용적률</td>
              <td>{{ popupData.foa }}%</td>
            </tr>
            <tr>
              <td>높이</td>
              <td>{{ popupData.height }}m</td>
            </tr>
            <tr>
              <td>면적</td>
              <td>{{ popupData.area }}㎡</td>
            </tr>
            <tr>
              <td>건축면적</td>
              <td>{{ popupData.aob }}㎡</td>
            </tr>
            <tr>
              <td>용적률 산정용 연면적</td>
              <td>{{ popupData.sof }}㎡</td>
            </tr>
          </tbody>
        </v-table>
      </v-card>
      <v-card v-if="popup2" style="position:absolute; top:200px; right: 400px" elevation="10">
        <v-card width="200">



          <v-card-title class="text-caption d-flex">그림자 설정
            <v-spacer></v-spacer>

          </v-card-title>
          <v-slider @update:model-value="changeMainLightPosition()" v-model="lightPosition" color="cyan"
            track-color="blue" class="mx-6" max="3.14" min="0"></v-slider>
          <v-slider v-model="shadowZ" color="cyan" track-color="blue" class="mx-6" max="10" min="-10"></v-slider>
        </v-card>
      </v-card>
      <v-card v-if="popup3" :style="'position:absolute; top: ' + (mousePosY + 7) + 'px; left: ' + (mousePosX + 20) + 'px'"
        elevation="10">
        <v-card>
          <v-card-title>
            {{ popup3Data }} 층
          </v-card-title>
        </v-card>
      </v-card>

      <v-card v-if="popup5" style="position:absolute; top:140px; right: 400px" elevation="10" max-width="500">
        <v-card flat>
          <v-card-title class="d-flex align-center">
            <strong>광원 정보</strong> <v-spacer></v-spacer> <v-btn
              @click="
                controlValue.visible = false;
              // controlValue.attach(lightsList[5])
              popup5 = false
                                                                                                                                                                                            "
              variant="text" icon="mdi-close"></v-btn>
          </v-card-title>
        </v-card>
        <v-btn-toggle v-model="lightType[selectedLightNo]">
          <v-btn @click="changeLightType()">
            <v-icon>mdi-power-off</v-icon>
          </v-btn>
          <v-btn @click="changeLightType()">
            <v-icon>mdi-lightbulb-on-outline </v-icon>
          </v-btn>
          <v-btn @click="changeLightType()">
            <v-icon>mdi-spotlight-beam</v-icon>
          </v-btn>
          <v-btn @click="changeLightType()">
            <v-icon>mdi-weather-sunny</v-icon>
          </v-btn>
          <v-btn @click="changeLightType()">
            <v-icon>mdi-ceiling-light</v-icon>
          </v-btn>
          <v-btn @click="changeLightType()">
            <v-icon>mdi-arrow-expand-right</v-icon>
          </v-btn>


        </v-btn-toggle>
        <v-color-picker elevation="0" hide-inputs v-model="selectedLightColor"
          @update:modelValue="changeLightColor()"></v-color-picker>
        <v-card-subtitle class="text-black font-weight-bold">
          빛 강도
        </v-card-subtitle>

        <v-slider thumb-label="always" v-model="selectedLight.intensity" color="#5338EB" track-color="blue" class="mx-6"
          max="5" min="0"></v-slider>

      </v-card>
      <v-card v-if="popup6" style="position:absolute; top:140px; right: 400px" elevation="10" max-width="500">
        <v-card flat>
          <v-card-title class="d-flex align-center">
            <strong>메쉬 정보</strong> <v-spacer></v-spacer> <v-btn @click="


              popup6 = false
            " variant="text" icon="mdi-close"></v-btn>
          </v-card-title>
        </v-card>
        <v-card-subtitle class="text-black font-weight-bold">
          메쉬 컬러
        </v-card-subtitle>
        <v-color-picker elevation="0" hide-inputs v-model="selectedMeshColor"
          @update:modelValue="changeMeshColor()"></v-color-picker>
        <v-card-subtitle class="text-black font-weight-bold" v-if="0 < hoverAction">
          메쉬선택 컬러
        </v-card-subtitle>
        <v-color-picker elevation="0" hide-inputs v-model="selectedMeshHoverColor"
          @update:modelValue="changeMeshHoverColor()" v-if="0 < hoverAction"></v-color-picker>
        <v-card-subtitle class="text-black font-weight-bold">
          투명도
        </v-card-subtitle>
        <v-slider thumb-label="always" v-model="selectedMeshOpacity" color="#5338EB" track-color="blue" class="mx-6"
          max="1" min="0"></v-slider>
        <v-card-subtitle class="text-black font-weight-bold">
          표면질감
        </v-card-subtitle>
        <v-slider thumb-label="always" v-model="selectedMeshShininess" color="#5338EB" track-color="blue" class="mx-6"
          max="100" min="0"></v-slider>
      </v-card>
    </v-main>
  </v-app>
</template>

<style>
::-webkit-scrollbar {
  width: 0px;
  background: transparent;
}

.three3d {
  width: 100%;
  height: 100%;
}

.obit-controls-gizmo {
  position: absolute;
  top: 15px;
  right: 420px;
  z-index: 1000;
  background-color: #FFF0;
  border-radius: 100%;
  transition: background-color .15s linear;
  cursor: pointer;
}

.obit-controls-gizmo.dragging,
.obit-controls-gizmo:hover {
  background-color: #FFF3;
}

.obit-controls-gizmo.inactive {
  pointer-events: none;
  background-color: #FFF0 !important;
}

.v-overlay__scrim.bg-false {
  background: transparent;

}
</style>

