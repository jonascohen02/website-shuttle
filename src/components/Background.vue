<template>
  <div id="container" ref="container"></div>
</template>

<script>
import * as THREE from "three";
import { ref, onMounted } from "vue";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import {
  Lensflare,
  LensflareElement,
} from "three/examples/jsm/objects/Lensflare.js";

// Variables
const radius = 6371;
const tiltEarth = 0.41;
const tiltMoon = (6.68 * 2 * Math.PI) / 360;
const rotationSpeed = 0.02;

const initLon = (-90 * Math.PI) / 180;
const initLat = (45 * Math.PI) / 180;
const initAlt = 15000;

var positionCamera = new THREE.Vector3();
positionCamera.setFromSphericalCoords(initAlt, initLat - 0.001, initLon);
var sphCam = new THREE.Spherical();

var positionShuttle = new THREE.Vector3();
positionShuttle.setFromSphericalCoords(initAlt - 100, initLat, initLon);
var sphShuttle = new THREE.Spherical();

const cloudsScale = 1.005;
const moonScale = 0.23;

const MARGIN = 0;
let SCREEN_HEIGHT = window.innerHeight - MARGIN * 2;
let SCREEN_WIDTH = window.innerWidth;

let camera, controls, scene, renderer;
let geometry, meshPlanet, meshClouds, meshMoon;
let shuttle;

const textureLoader = new THREE.TextureLoader();

const clock = new THREE.Clock();

export default {
  emits: ["background-has-loaded"],
  props: {
    isGameEnable: Boolean,
  },
  setup(props, { emit }) {
    const container = ref(null);

    let backgroundProgression = function (obj) {
      emit("background-has-loaded", obj);
    };

    onMounted(async () => {
      init();
      animate();

      // Functions
      function init() {
        // scene
        scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x000000, 0.00000025);

        //camera
        camera = new THREE.PerspectiveCamera(
          45,
          SCREEN_WIDTH / SCREEN_HEIGHT,
          10,
          1000000
        );

        // convert and update the variable containing the position
        camera.position.set(
          positionCamera.x,
          positionCamera.y,
          positionCamera.z
        );

        scene.add(camera);

        // earth texture
        const materialNormalMap = new THREE.MeshPhongMaterial({
          specular: 0x333333,
          shininess: 15,
          map: textureLoader.load("textures/earth_atmos_2048.jpg"),
          specularMap: textureLoader.load("textures/earth_specular_2048.jpg"),
          normalMap: textureLoader.load("textures/earth_normal_2048.jpg"),

          // y scale is negated to compensate for normal map handedness.
          normalScale: new THREE.Vector2(0.85, -0.85),
        });

        // sun
        const light = new THREE.AmbientLight(0x404040, 1); // soft white light
        scene.add(light);

        const textureFlare0 = textureLoader.load("textures/lensflare0.png");
        const sun = new THREE.PointLight(0xffffff, 2);
        sun.color.setHSL(0.995, 0.5, 0.9);
        sun.position.set(0, 0, radius * 20);

        const lensflare = new Lensflare();
        lensflare.addElement(
          new LensflareElement(textureFlare0, 700, 0, sun.color)
        );
        sun.add(lensflare);
        scene.add(sun);

        backgroundProgression("Sun");

        // earth
        geometry = new THREE.SphereBufferGeometry(radius, 100, 50);

        meshPlanet = new THREE.Mesh(geometry, materialNormalMap);
        meshPlanet.rotation.y = -Math.PI;
        meshPlanet.rotation.z = tiltEarth;
        meshPlanet.name = "Earth";
        scene.add(meshPlanet);

        // clouds
        const materialClouds = new THREE.MeshLambertMaterial({
          map: textureLoader.load("textures/earth_clouds_1024.png"),
          transparent: true,
        });

        meshClouds = new THREE.Mesh(geometry, materialClouds);
        meshClouds.scale.set(cloudsScale, cloudsScale, cloudsScale);
        meshClouds.rotation.z = tiltEarth;
        meshClouds.name = "Earth's clouds";
        scene.add(meshClouds);

        backgroundProgression("Earth");

        // moon
        const materialMoon = new THREE.MeshPhongMaterial({
          map: textureLoader.load("textures/moon_1024.jpg"),
        });

        meshMoon = new THREE.Mesh(geometry, materialMoon);
        meshMoon.rotation.z = tiltMoon;
        meshMoon.position.set(radius * 5, 0, radius * 5);
        meshMoon.scale.set(moonScale, moonScale, moonScale);
        meshMoon.name = "Moon";
        scene.add(meshMoon);

        backgroundProgression("Moon");

        // stars
        const r = radius,
          starsGeometry = [
            new THREE.BufferGeometry(),
            new THREE.BufferGeometry(),
          ];

        const vertices1 = [];
        const vertices2 = [];

        const vertex = new THREE.Vector3();

        for (let i = 0; i < 250; i++) {
          vertex.x = Math.random() * 2 - 1;
          vertex.y = Math.random() * 2 - 1;
          vertex.z = Math.random() * 2 - 1;
          vertex.multiplyScalar(r);

          vertices1.push(vertex.x, vertex.y, vertex.z);
        }

        for (let i = 0; i < 1500; i++) {
          vertex.x = Math.random() * 2 - 1;
          vertex.y = Math.random() * 2 - 1;
          vertex.z = Math.random() * 2 - 1;
          vertex.multiplyScalar(r);

          vertices2.push(vertex.x, vertex.y, vertex.z);
        }

        starsGeometry[0].setAttribute(
          "position",
          new THREE.Float32BufferAttribute(vertices1, 3)
        );
        starsGeometry[1].setAttribute(
          "position",
          new THREE.Float32BufferAttribute(vertices2, 3)
        );

        const starsMaterials = [
          new THREE.PointsMaterial({
            color: 0x555555,
            size: 2,
            sizeAttenuation: false,
          }),
          new THREE.PointsMaterial({
            color: 0x555555,
            size: 1,
            sizeAttenuation: false,
          }),
          new THREE.PointsMaterial({
            color: 0x333333,
            size: 2,
            sizeAttenuation: false,
          }),
          new THREE.PointsMaterial({
            color: 0x3a3a3a,
            size: 1,
            sizeAttenuation: false,
          }),
          new THREE.PointsMaterial({
            color: 0x1a1a1a,
            size: 2,
            sizeAttenuation: false,
          }),
          new THREE.PointsMaterial({
            color: 0x1a1a1a,
            size: 1,
            sizeAttenuation: false,
          }),
        ];

        for (let i = 10; i < 30; i++) {
          const stars = new THREE.Points(
            starsGeometry[i % 2],
            starsMaterials[i % 6]
          );

          stars.rotation.x = Math.random() * 6;
          stars.rotation.y = Math.random() * 6;
          stars.rotation.z = Math.random() * 6;
          stars.scale.setScalar(i * 10);

          stars.matrixAutoUpdate = false;
          stars.updateMatrix();

          scene.add(stars);
        }

        backgroundProgression("Stars");

        shuttle;

        const loader = new GLTFLoader().setPath("model/shuttle/");
        loader.load(
          "scene.gltf",
          // success
          function (gltf) {
            shuttle = gltf.scene;

            // convert and update the variable containing the position
            shuttle.position.set(
              positionShuttle.x,
              positionShuttle.y,
              positionShuttle.z
            );

            // rotation
            shuttle.rotation.y = Math.PI;
            scene.add(shuttle);

            controls.target = shuttle.position;

            backgroundProgression("Shuttle");
          },
          // called while loading is progressing
          function () {},
          function (error) {
            console.error(error);
          }
        );

        // renderer
        renderer = new THREE.WebGLRenderer({
          antialiasing: true,
        });
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);
        container.value.appendChild(renderer.domElement);

        // Controls
        controls = new OrbitControls(camera, document.getElementById("container"));

        window.addEventListener("resize", onWindowResize, false);
      }

      function onWindowResize() {
        SCREEN_HEIGHT = window.innerHeight;
        SCREEN_WIDTH = window.innerWidth;

        camera.aspect = SCREEN_WIDTH / SCREEN_HEIGHT;
        camera.updateProjectionMatrix();

        renderer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);
      }

      function animate() {
        requestAnimationFrame(animate);

        controls.update();

        render();
      }

      function render() {
        // rotate the planet and clouds
        const delta = clock.getDelta();

        meshPlanet.rotation.y += rotationSpeed * delta;
        meshClouds.rotation.y += 1.25 * rotationSpeed * delta;

        // get current location
        positionCamera = camera.position;
        // edit the associated spherical vector
        sphCam.setFromVector3(positionCamera);
        // edit the theta angle at each frame for faking a movement
        sphCam.theta += 0.001;
        // convert and update the variable containing the position
        positionCamera.setFromSpherical(sphCam);
        // update cam position
        camera.position.set(
          positionCamera.x,
          positionCamera.y,
          positionCamera.z
        );

        // same for shuttle
        if (shuttle) {
          positionShuttle = shuttle.position;
          sphShuttle.setFromVector3(positionShuttle);
          sphShuttle.theta += 0.001;
          positionShuttle.setFromSpherical(sphShuttle);
          shuttle.position.set(
            positionShuttle.x,
            positionShuttle.y,
            positionShuttle.z
          );
          shuttle.rotation.y += 0.001;
          shuttle.rotation.z += 0.01;
        }

        renderer.render(scene, camera);
      }

      // let max_y = 25;
      // let min_y = -25;
      // let max_x = 25;
      // let min_x = -25;

      // // game start
      // watch(
      //   () => props.isGameEnable,
      //   () => {
      //     if (props.isGameEnable) {
      //       stopRotationShuttle();
      //       // set shuttle in center and correct orientation
      //       shuttle.rotation.z = 0;

      //       // camera
      //       // TODO
      //       // camera.position.set(0, -50, 50);

      //       // change controls
      //       controls.enabled = false;

      //       // if tactile device
      //       document.addEventListener("touchmove", function (e) {
      //         repositionShuttle(e);
      //         // TODO : block scroll on mobile
      //       });

      //       // if computer
      //       document.addEventListener("mousemove", function (e) {
      //         repositionShuttle(e);
      //       });
      //       // load obstacles

      //       // make them go toward viewer

      //       // if contact, lose

      //       // if not, keep going
      //     }
      //   }
      // );
    });

    return {
      container,
    };
  },
};
</script>

<style>
</style>