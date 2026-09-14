<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <script src="https://aframe.io/releases/1.4.0/aframe.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/mind-ar@1.2.5/dist/mindar-image-aframe.prod.js"></script>
  <script>
    // Yeşil ekranı (chroma key) şeffaf yapan özel shader
    AFRAME.registerShader('chromakey', {
      schema: {
        src: {type: 'map', is: 'uniform'},
        color: {type: 'color', is: 'uniform', default: '#00FF00'},
        threshold: {type: 'number', is: 'uniform', default: 0.45},
        smoothing: {type: 'number', is: 'uniform', default: 0.08}
      },
      vertexShader: `
        varying vec2 vUv;
        void main() {
          vUv = uv;
          gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
        }
      `,
      fragmentShader: `
        uniform sampler2D src;
        uniform vec3 color;
        uniform float threshold;
        uniform float smoothing;
        varying vec2 vUv;
        void main() {
          vec4 texColor = texture2D(src, vUv);
          float diff = length(texColor.rgb - color);
          float alpha = smoothstep(threshold, threshold + smoothing, diff);
          gl_FragColor = vec4(texColor.rgb, alpha);
        }
      `
    });

    // Titremeyi azaltan yumuşatma (lerp) bileşeni
    AFRAME.registerComponent('smooth-anchor', {
      schema: {speed: {type: 'number', default: 0.25}},
      init: function () {
        this.targetPos = new THREE.Vector3();
        this.targetQuat = new THREE.Quaternion();
        this.initialized = false;
      },
      tick: function () {
        const obj = this.el.object3D;
        if (!this.initialized) {
          this.targetPos.copy(obj.position);
          this.targetQuat.copy(obj.quaternion);
          this.initialized = true;
          return;
        }
        this.targetPos.lerp(obj.position, this.data.speed);
        this.targetQuat.slerp(obj.quaternion, this.data.speed);
      }
    });
  </script>
</head>
<body style="margin:0; overflow:hidden;">

  <a-scene
    mindar-image="imageTargetSrc: ./targets.mind; filterMinCF:0.00001; filterBeta: 1; missTolerance:15; warmupTolerance:3;"
    color-space="sRGB"
    renderer="colorManagement: true; physicallyCorrectLights: true; alpha: true;"
    vr-mode-ui="enabled: false"
    device-orientation-permission-ui="enabled: true"
  >
    <a-assets>
      <video id="personVideo" src="./person.mp4" autoplay loop muted playsinline crossorigin="anonymous"></video>
    </a-assets>

    <a-camera position="0 0 0" look-controls="enabled: false" cursor="fuse: false"></a-camera>

    <a-entity mindar-image-target="targetIndex: 0">
      <!-- Figür: kapının hemen yanında, yeşil ekran temizlenmiş, şeffaf arka planlı -->
      <a-plane
        material="shader: chromakey; src: #personVideo; transparent: true; side: double"
        position="0.15 0.25 0"
        rotation="0 0 0"
        width="0.45"
        height="0.65"
      ></a-plane>
    </a-entity>

  </a-scene>

</body>
</html>
