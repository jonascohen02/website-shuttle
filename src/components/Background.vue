<template>
  <div id="container" ref="container"></div>
</template>

<script>
import * as THREE from "three";
import { ref, onMounted, watch } from "vue";
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
const rotationSpeed = 0.0005;

const initLon = (-90 * Math.PI) / 180;
const initLat = (45 * Math.PI) / 180;
const initAlt = 15000;

var positionCamera = new THREE.Vector3();
var sphCam = new THREE.Spherical();
sphCam.set(initAlt, initLat, initLon);
positionCamera.setFromSpherical(sphCam);

var positionShuttle = new THREE.Vector3();
var sphShuttle = new THREE.Spherical();
sphShuttle.set(initAlt - 100, initLat, initLon);
positionShuttle.setFromSpherical(sphShuttle);

var positionGroup = new THREE.Vector3();
var sphGroup = new THREE.Spherical();
sphGroup.set(6700, initLat, initLon);
positionGroup.setFromSpherical(sphGroup);

const speed_shuttle = 0.001;
const speed_rotation_shuttle = 0.01;

const cloudsScale = 1.005;
const moonScale = 0.23;

const MARGIN = 0;
let SCREEN_HEIGHT = window.innerHeight - MARGIN * 2;
let SCREEN_WIDTH = window.innerWidth;

let camera, controls, scene, renderer;
let geometry, meshPlanet, meshClouds, meshMoon;
let shuttle, group, camera_pivot, obstacle, shuttleIsTouched;

const radiusObstacle = 5;
var obstacleSpeed = 5;
var score = 0;
var gameJustStarted;
var timeOutTriggered = false;

var userDidNotSeeRules = true;

const textureLoader = new THREE.TextureLoader();

export default {
  emits: ["backgroundhasloaded", "stopgame"],
  props: {
    isGameEnable: Boolean,
  },
  setup(props, { emit }) {
    const container = ref(null);

    let backgroundProgression = function (obj) {
      emit("backgroundhasloaded", obj);
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

        const light = new THREE.AmbientLight(0x404040, 1); // soft white light
        scene.add(light);

        const earthLight = new THREE.PointLight(0xffffff, 1);
        earthLight.color.setHSL(0.995, 0.5, 0.9);
        earthLight.position.set(0, 0, radius);
        scene.add(earthLight);

        // sun
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

        // shuttle
        const loader = new GLTFLoader().setPath("models/shuttle/");
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

        const loaderAsteroid = new GLTFLoader().setPath("models/asteroid/");
        loaderAsteroid.load(
          "scene.gltf",
          // success
          function (gltf) {
            obstacle = gltf.scene;
            obstacle.scale.set(0.015, 0.015, 0.015);
          },
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
        controls = new OrbitControls(
          camera,
          document.getElementById("container")
        );

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
        const delta = 1;

        meshPlanet.rotation.y += rotationSpeed * delta;
        meshClouds.rotation.y += 1.25 * rotationSpeed * delta;

        // check if shuttle exists
        if (shuttle) {
          // normal mode (home page)
          if (!props.isGameEnable) {
            // get current location
            positionCamera = camera.position;
            // edit the associated spherical vector
            sphCam.setFromVector3(positionCamera);
            // edit the theta angle at each frame for faking a movement
            sphCam.theta += speed_shuttle;
            // convert and update the variable containing the position
            positionCamera.setFromSpherical(sphCam);
            // update cam position
            camera.position.set(
              positionCamera.x,
              positionCamera.y,
              positionCamera.z
            );

            // orientation shuttle along its long axis
            shuttle.rotation.z += speed_rotation_shuttle;

            // same for the shuttle
            positionShuttle = shuttle.position;
            sphShuttle.setFromVector3(positionShuttle);
            sphShuttle.theta += speed_shuttle;
            positionShuttle.setFromSpherical(sphShuttle);
            shuttle.position.set(
              positionShuttle.x,
              positionShuttle.y,
              positionShuttle.z
            );
            shuttle.rotation.y += speed_shuttle;

            // game mode
          } else {
            // if in game, update group position
            positionGroup = group.position;
            sphGroup.setFromVector3(positionGroup);
            sphGroup.theta += speed_shuttle;
            positionGroup.setFromSpherical(sphGroup);
            group.position.set(
              positionGroup.x,
              positionGroup.y,
              positionGroup.z
            );
            group.rotation.y += speed_shuttle;

            // message to explain rules before sending obstacles
            if (userDidNotSeeRules) {
              if (!timeOutTriggered) {
                timeOutTriggered = true;
                setTimeout(() => {
                  alert("Stay alive, avoid the meteorites.");
                  userDidNotSeeRules = false;
                  timeOutTriggered = false;
                  createObstacles();
                }, 3000);
              }
            }

            // obstacle can exist only if user saw the rules once
            if (obstacle) {
              if (gameJustStarted) {
                if (!timeOutTriggered) {
                  timeOutTriggered = true;
                  setTimeout(() => {
                    group.add(obstacle);
                    gameJustStarted = false;
                    timeOutTriggered = false;
                    obstacle.position.z = 200;
                  }, 1000);
                }
              } else {
                // check if intersection with shuttle
                if (!shuttleIsTouched && Math.abs(obstacle.position.z) < 10) {
                  let distance = Math.pow(
                    Math.pow(shuttle.position.x - obstacle.position.x, 2) +
                      Math.pow(shuttle.position.y - obstacle.position.y, 2),
                    0.5
                  );

                  // if distance between shuttle and obstacle's center is less than 2 * obstacle's radius, there is impact, stop game
                  if (distance < 2 * radiusObstacle) {
                    shuttleIsTouched = true;
                    group.remove(obstacle);
                    setTimeout(() => {
                      emit("stopgame");
                    }, 2000);
                  }
                }

                // put it back at front if behind camera
                if (obstacle.position.z < -100) {
                  obstacle.position.z = 200;
                  obstacle.position.x = 0.5 * getRandomArbitrary(min_x, max_x);
                  obstacle.position.y = 0.5 * getRandomArbitrary(min_y, max_y);
                  score++;
                }
                // make it come towards the camera
                obstacle.position.z -= obstacleSpeed;
                // increase the speed of asteroid
                obstacleSpeed += 0.003;

                // make effects if shuttle is touched
                if (shuttleIsTouched) {
                  shuttle.rotation.y += 0.1;
                }
              }
            }
          }
        }

        renderer.render(scene, camera);
      }

      let max_y = 50;
      let min_y = -50;
      let max_x = 25;
      let min_x = -25;

      let repositionShuttle = function (e) {
        // avoid scroll refresh in android chrome
        e.preventDefault();

        // 2 ways to get pageX (depends on devices)
        let pageX = e.pageX || e.changedTouches[0].pageX;
        let pageY = e.pageY || e.changedTouches[0].pageY;
        shuttle.position.x = -(
          min_x +
          (pageX / SCREEN_WIDTH) * (max_x - min_x)
        );
        shuttle.position.y = max_y - (pageY / SCREEN_HEIGHT) * (max_y - min_y);
      };

      let getRandomArbitrary = function (min, max) {
        return Math.random() * (max - min) + min;
      };

      let createObstacles = function () {
        // realistic asteroids (requires good hardware)
        group.add(obstacle);
        obstacle.position.set(0, 0, 200);

        // spheric simple obstacles
        // const geometryObstacle = new THREE.SphereGeometry(
        //   radiusObstacle,
        //   32,
        //   32
        // );
        // const materialObstacle = new THREE.MeshBasicMaterial({
        //   color: "grey",
        // });
        // obstacle = new THREE.Mesh(geometryObstacle, materialObstacle);
        // group.add(obstacle);
        // obstacle.position.set(0, 0, 200);
      };

      // game start
      watch(
        () => props.isGameEnable,
        () => {
          if (props.isGameEnable) {
            group = new THREE.Group();
            group.add(shuttle);

            // camera
            camera_pivot = new THREE.Group();
            group.add(camera_pivot);

            // group position
            sphGroup.set(6700, initLat, initLon);
            positionGroup.setFromSpherical(sphGroup);
            group.position.set(
              positionGroup.x,
              positionGroup.y,
              positionGroup.z
            );

            // set shuttle in center and correct orientation
            shuttle.rotation.z = 0;
            shuttle.rotation.y = Math.PI;
            shuttle.position.set(0, 0, 0);

            camera_pivot.rotation.y = 0.15;
            camera_pivot.rotation.x = 0.1;

            camera_pivot.position.set(0, 0, 0);
            camera_pivot.add(camera);
            camera.position.set(0, 0, -100);

            controls.target = group.position;

            // disable controls
            controls.enabled = false;

            // if tactile device
            document.addEventListener("touchmove", repositionShuttle, {
              passive: false,
            });

            // if computer
            document.addEventListener("mousemove", repositionShuttle, {
              passive: false,
            });

            scene.add(group);

            // var initialization
            shuttleIsTouched = false;
            score = 0;
            gameJustStarted = true;
          } else {
            document.removeEventListener("mousemove", repositionShuttle);
            document.removeEventListener("mousemove", repositionShuttle);

            alert("Game Over. \n\nScore : " + score.toString());

            // remove group and come back to home mode
            group.remove(shuttle);
            group.remove(camera_pivot);
            camera_pivot.remove(camera);
            scene.remove(group);

            scene.add(camera);
            scene.add(shuttle);

            // set objects'position & orientation
            sphCam.set(initAlt, initLat, initLon);
            positionCamera.setFromSpherical(sphCam);
            camera.position.set(
              positionCamera.x,
              positionCamera.y,
              positionCamera.z
            );

            sphShuttle.set(initAlt - 100, initLat, initLon);
            positionShuttle.setFromSpherical(sphShuttle);
            shuttle.position.set(
              positionShuttle.x,
              positionShuttle.y,
              positionShuttle.z
            );
            shuttle.rotation.y = Math.PI;

            controls.target = shuttle.position;
            controls.enabled = true;
          }
        }
      );
    });

    return {
      container,
    };
  },
};
</script>

<style>
</style>