<template>
  <div>
    <div>
      <div id="container"></div>
    </div>
  </div>
</template>

<script>
import THREE from "./libs/three";
import vox from "vox.js";

export default {
  name: "VoxTest",
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      mesh: null,
      light1: null,
      light2: null,
      light3: null,
      light4: null,
      clock: new THREE.Clock(),
      voxelModelNamesIndex: 0,
      voxelModelNames: [
        "garage-94-delorean",
        "newspaper1",
        "garage-1-robotarm-3",
        "garage-88-floor"
      ],
      voxelModels: {}
    };
  },
  methods: {
    init: function() {
      let container = document.getElementById("container");

      this.camera = new THREE.PerspectiveCamera(
        45,
        container.clientWidth / container.clientHeight,
        0.01,
        1000
      );
      this.camera.position.z = 800;
      this.camera.position.y = 400;

      this.scene = new THREE.Scene();

      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(container.clientWidth, container.clientHeight);
      container.appendChild(this.renderer.domElement);

      var controls = new THREE.OrbitControls(
        this.camera,
        this.renderer.domElement
      );
      controls.screenSpacePanning = true;
    },
    animate() {
      requestAnimationFrame(this.animate);
      // Anything after to animate

      this.render();
    },
    render() {
      var time = Date.now() * 0.0005;
      this.light1.position.x = Math.sin(time * 0.7) * 30;
      this.light1.position.y = Math.cos(time * 0.5) * 40;
      this.light1.position.z = Math.cos(time * 0.3) * 30;
      this.light2.position.x = Math.cos(time * 0.3) * 30;
      this.light2.position.y = Math.sin(time * 0.5) * 40;
      this.light2.position.z = Math.sin(time * 0.7) * 30;
      this.light3.position.x = Math.sin(time * 0.7) * 30;
      this.light3.position.y = Math.cos(time * 0.3) * 40;
      this.light3.position.z = Math.sin(time * 0.5) * 30;
      this.light4.position.x = Math.sin(time * 0.3) * 30;
      this.light4.position.y = Math.cos(time * 0.7) * 40;
      this.light4.position.z = Math.sin(time * 0.5) * 30;

      if (this.voxelModels.hasOwnProperty("garage-94-delorean")) {
        // this.voxelModels["garage-94-delorean"].position.x += 2;
      }

      this.renderer.render(this.scene, this.camera);
    },
    addLight() {
      let hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 0.6);
      hemiLight.color.setHSL(0.6, 1, 0.6);
      hemiLight.groundColor.setHSL(0.095, 1, 0.75);
      hemiLight.position.set(0, 50, 0);
      this.scene.add(hemiLight);

      let sphere = new THREE.SphereBufferGeometry(0.5, 16, 8);

      this.light1 = new THREE.PointLight(0xff0040, 2, 50);
      this.light1.add(
        new THREE.Mesh(sphere, new THREE.MeshBasicMaterial({ color: 0xff0040 }))
      );
      this.scene.add(this.light1);
      this.light2 = new THREE.PointLight(0x0040ff, 2, 50);
      this.light2.add(
        new THREE.Mesh(sphere, new THREE.MeshBasicMaterial({ color: 0x0040ff }))
      );
      this.scene.add(this.light2);
      this.light3 = new THREE.PointLight(0x80ff80, 2, 50);
      this.light3.add(
        new THREE.Mesh(sphere, new THREE.MeshBasicMaterial({ color: 0x80ff80 }))
      );
      this.scene.add(this.light3);
      this.light4 = new THREE.PointLight(0xffaa00, 2, 50);
      this.light4.add(
        new THREE.Mesh(sphere, new THREE.MeshBasicMaterial({ color: 0xffaa00 }))
      );
      this.scene.add(this.light4);
    },
    addHelper() {
      var helper = new THREE.GridHelper(160, 10);
      helper.rotation.x = Math.PI / 2;
      this.scene.add(helper);
    },
    addTestMesh() {
      let geometry = new THREE.BoxGeometry(0.2, 0.2, 0.2);
      let material = new THREE.MeshNormalMaterial();

      this.mesh = new THREE.Mesh(geometry, material);
      this.scene.add(this.mesh);
    },
    addTestFloor() {
      // Geometry
      var cbgeometry = new THREE.PlaneGeometry(500, 500, 8, 8);

      // Materials
      var cbmaterials = [];

      cbmaterials.push(
        new THREE.MeshBasicMaterial({ color: 0xffffff, side: THREE.DoubleSide })
      );
      cbmaterials.push(
        new THREE.MeshBasicMaterial({ color: 0x000000, side: THREE.DoubleSide })
      );

      var l = cbgeometry.faces.length / 2; // <-- Right here. This should still be 8x8 (64)

      for (var i = 0; i < l; i++) {
        let j = i * 2; // <-- Added this back so we can do every other 'face'
        cbgeometry.faces[j].materialIndex = (i + Math.floor(i / 8)) % 2; // The code here is changed, replacing all 'i's with 'j's. KEEP THE 8
        cbgeometry.faces[j + 1].materialIndex = (i + Math.floor(i / 8)) % 2; // Add this line in, the material index should stay the same, we're just doing the other half of the same face
      }

      // Mesh
      var cb = new THREE.Mesh(
        cbgeometry,
        new THREE.MeshFaceMaterial(cbmaterials)
      );
      cb.rotation.x = Math.PI / 2;
      cb.position.y = -1.5;

      this.scene.add(cb);
    },
    loadCurrentVoxel() {
      if (
        typeof this.voxelModelNames[this.voxelModelNamesIndex] !== "undefined"
      ) {
        var parser = new vox.Parser();
        var modelName = this.voxelModelNames[this.voxelModelNamesIndex];
        parser.parse(`models/${modelName}.vox`).then(voxelData => {
          var param = { voxelSize: 1 };
          var builder = new vox.MeshBuilder(voxelData, param);
          var mesh = builder.createMesh();

          this.scene.add(mesh);
          this.voxelModels[modelName] = mesh;
          this.voxelModelNamesIndex++;
          this.loadCurrentVoxel();
        });
      } else {
        this.onVoxelLoadComplete();
      }
    },
    onVoxelLoadComplete() {
      this.placeVoxelModels();
    },
    placeVoxelModels() {
      let newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(0, 0, 0); // or any other coordinates
      this.scene.add(newfloor);

      newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(126, 0, 0); // or any other coordinates
      this.scene.add(newfloor);

      newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(-126, 0, 0); // or any other coordinates
      this.scene.add(newfloor);

      newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(0, 0, 126); // or any other coordinates
      this.scene.add(newfloor);

      newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(126, 0, 126); // or any other coordinates
      this.scene.add(newfloor);

      newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(-126, 0, 126); // or any other coordinates
      this.scene.add(newfloor);

      newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(0, 0, -126); // or any other coordinates
      this.scene.add(newfloor);

      newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(126, 0, -126); // or any other coordinates
      this.scene.add(newfloor);

      newfloor = this.voxelModels["garage-88-floor"].clone();
      newfloor.position.set(-126, 0, -126); // or any other coordinates
      this.scene.add(newfloor);

      // var newfloor4 = this.voxelModels["garage-88-floor"].clone();
      // newfloor4.position.set(0,0,260); // or any other coordinates
      // this.scene.add(newfloor4);
    }
  },
  mounted() {
    this.init();

    this.addTestFloor();

    this.addLight();

    // this.addTestMesh();
    this.addHelper();

    this.loadCurrentVoxel();

    // this.loadVoxel("newspaper1", { x: 1, y: 20, z: 10 });
    // this.loadVoxel("garage-1-robotarm-3", { x: 1, y: 20, z: 10 });
    this.animate();
  }
};
</script>

<style>
body {
  padding: 0;
  margin: 0;
}

#container {
  width: 100%;
  height: 800px;
}
</style>
