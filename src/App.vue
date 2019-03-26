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
      mesh: null
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
    animate: function() {
      this.renderer.render(this.scene, this.camera);
      requestAnimationFrame(this.animate);

      // Anything after to animate
    },
    addLight() {
      var light = new THREE.AmbientLight(0xffffff); // soft white light
      this.scene.add(light);
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
    loadVoxel(modelName, position, material) {
      var parser = new vox.Parser();
      parser.parse(`models/${modelName}.vox`).then(voxelData => {
        voxelData.voxels; // voxel position and color data
        voxelData.size; // model size
        voxelData.palette; // palette data

        var param = { voxelSize: 1 };
        var builder = new vox.MeshBuilder(voxelData, param);
        var mesh = builder.createMesh();

        if (material) {
          material(mesh);
        }

        if (position) {
          mesh.position.x = position.x;
          mesh.position.y = position.y;
          mesh.position.z = position.z;
        }

        this.scene.add(mesh);
      });
    },
    oilTexture(mesh) {
      let uniforms = {
        u_time: { type: "f", value: 1.0 },
        u_resolution: { type: "v2", value: new THREE.Vector2() },
        u_mouse: { type: "v2", value: new THREE.Vector2() }
      };

      let material = new THREE.ShaderMaterial({
        uniforms: uniforms,
        vertexShader: document.getElementById("vertexShader").textContent,
        fragmentShader: document.getElementById("fragmentShader").textContent
      });

      mesh.material = material;
    }
  },
  mounted() {
    this.init();

    this.addTestFloor();

    this.addLight();

    // this.addTestMesh();
    this.addHelper();

    this.loadVoxel("garage-84-oil", { x: 1, y: 0, z: 0 }, this.oilTexture);
    this.loadVoxel("newspaper1", { x: 1, y: 20, z: 10 });
    this.loadVoxel("garage-1-robotarm-3", { x: 1, y: 20, z: 10 });
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
