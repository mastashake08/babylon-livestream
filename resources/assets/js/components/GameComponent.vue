<template>
  <canvas id="renderCanvas"></canvas>
</template>

<script>
    export default {
        mounted() {
            console.log('Component mounted.')

              this.createGameEnivroment();
        },
        data(){
          return {
            canvas: {},
            engine: {},
            sphere: {},
            ground: {},
            groundMaterial: {},
            assetsManager: {},
            skybox: {},
            camera: {},
            worldWidth: 300,
            worldHeight: 300,
            worldDivisions: 256,
            sphereSegment: 16,
            sphereDiameter: 25
          }
        },
        created(){
        },
        methods: {
          createGameEnivroment: function(){
            this.canvas = document.getElementById('renderCanvas');
            this.engine = new BABYLON.Engine(this.canvas,true,{preserveDrawingBuffer: true, stencil: true});
            this.scene = this.createScene();
            this.skybox = this.createSkyBox();
            // run the render loop
            this.scene.registerBeforeRender(this.beforeRenderFunction());
            var that = this;
            this.engine.runRenderLoop(function(){
                that.scene.render();
            });
            // the canvas/window resize event handler
            window.addEventListener('resize', function(){
                that.engine.resize();
            });
          },
          createSkyBox: function(){
            var skybox = BABYLON.MeshBuilder.CreateBox("skyBox", {size:1000.0}, this.scene);
            var skyboxMaterial = new BABYLON.StandardMaterial("skyBox", this.scene);
            skyboxMaterial.backFaceCulling = false;
            skyboxMaterial.reflectionTexture = new BABYLON.CubeTexture("textures/skybox", this.scene);
            skyboxMaterial.reflectionTexture.coordinatesMode = BABYLON.Texture.SKYBOX_MODE;
            skyboxMaterial.diffuseColor = new BABYLON.Color3(0, 0, 0);
            skyboxMaterial.specularColor = new BABYLON.Color3(0, 0, 0);
            skybox.material = skyboxMaterial;
            return skybox;
          },
          createScene: function(){
            // Create a basic BJS Scene object
            var scene = new BABYLON.Scene(this.engine);
            // Create a FreeCamera, and set its position to {x: 0, y: 5, z: -10}
            this.camera = new BABYLON.FreeCamera('camera1', new BABYLON.Vector3(0, 5, -10), scene);
            // Target the camera to scene origin
            this.camera.setTarget(BABYLON.Vector3.Zero());
            // Attach the camera to the canvas
            this.camera.attachControl(this.canvas, false);
            // Create a basic light, aiming 0, 1, 0 - meaning, to the sky
            var light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene);
            // Create a built-in "sphere" shape; its constructor takes 6 params: name, segment, diameter, scene, updatable, sideOrientation
            this.sphere = BABYLON.Mesh.CreateSphere('sphere1', this.sphereSegment, this.sphereDiameter, scene, false, BABYLON.Mesh.FRONTSIDE);
            // Move the sphere upward 1/2 of its height
            this.sphere.position.y = this.sphereDiameter;
            this.sphere.receiveShadows = true;
            // Create a built-in "ground" shape; its constructor takes 6 params : name, width, height, subdivision, scene, updatable
            this.ground = BABYLON.Mesh.CreateGroundFromHeightMap("ground", "textures/heightmap.jpg", this.worldWidth, this.worldHeight, this.worldDivisions, 0, 10, scene, false, this.successCallback);
            this.createMaterial(scene);
            this.ground.receiveShadows = true;

            // Return the created scene
            return scene;
          },
          createMaterial: function(scene){
         this.groundMaterial = new BABYLON.StandardMaterial("ground", scene);
         this.groundMaterial.diffuseTexture = new BABYLON.Texture("textures/earth_land.jpg", scene);
         this.assetsManager = new BABYLON.AssetsManager(scene);

         var that = this;

          this.assetsManager.onProgress = function(remainingCount, totalCount, lastFinishedTask) {
            console.log("loading....");
              this.engine.loadingUIText = 'We are loading the scene. ' + remainingCount + ' out of ' + totalCount + ' items still need to be loaded.';
          };
          this.assetsManager.onFinish = function(tasks) {
              that.engine.runRenderLoop(function() {
                  scene.render();
              });
          };
          this.assetsManager.load();
         this.ground.material = this.groundMaterial;

        },
        successCallback: function(mesh){
        },
        beforeRenderFunction: function () {
                  // Camera
                  if (this.camera.beta < 0.1)
                      this.camera.beta = 0.1;
                  else if (this.camera.beta > (Math.PI / 2) * 0.9)
                      this.camera.beta = (Math.PI / 2) * 0.9;
                  if (this.camera.radius > 50)
                      this.camera.radius = 50;
                  if (this.camera.radius < 5)
                      this.camera.radius = 5;
              },
          },
        }
</script>
