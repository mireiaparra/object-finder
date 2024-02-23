<template>
  <div class="container">
    <div v-if="isLoadingModel">Loading Model ...</div>
    <div v-else>
      <label for="avatar">Choose a picture:</label>
      <input
        @change="getChange"
        type="file"
        id="avatar"
        name="avatar"
        accept="image/*"
      />
    </div>
    <div class="background">
      <div class="particle-1"></div>
      <div class="particle-2"></div>
      <div class="polaroid" >
        <div class="image-container">
          <img
            :class="imageSrc ? '' : 'display-none'"
            :src="imageSrc"
            ref="imageToDetect"
            :width="imageWidth"
            :height="imageHeight"
          />
          <p v-if="!imageSrc">Upload your photo!</p>
          <div
            v-for="(result, index) in results"
            :key="index"
            class="bounding-box"
            :style="{
              top: result.bbox[1] + 'px',
              left: result.bbox[0] + 'px',
              width: result.bbox[2] + 'px',
              height: result.bbox[3] + 'px',
              borderColor: colors[index % colors.length],
            }"
          >
            <span>{{ result.score.toFixed(3) }} {{ result.class }}</span>
          </div>
        </div>
        <div class="image-caption">{{ imageName }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watchEffect } from "vue";
import "@tensorflow/tfjs-backend-cpu";
import "@tensorflow/tfjs-backend-webgl";
import * as tf from "@tensorflow/tfjs";
import * as cocoSsd from "@tensorflow-models/coco-ssd";

// Reactive variables
const isLoadingModel = ref(false);
let cocoSsdModel = null;
const imageSrc = ref(null);
const imageToDetect = ref(null);
const imageName = ref("");
const imageWidth = ref(0);
const imageHeight = ref(0);

const colors = ["red", "green", "blue"];
const results = ref([]);

// Load the model when the component is mounted
onMounted(async () => {
  isLoadingModel.value = true;
  cocoSsdModel = await cocoSsd.load();
  isLoadingModel.value = false;
});

// Watch for changes in imageSrc and run object detection when it changes
watchEffect(() => {
  if (imageSrc.value) {
    getDetect();
  }
});

// Update imageSrc when a new file is selected
const getChange = (e) => {
  imageToDetect.value.onload = () => {
    imageWidth.value = imageToDetect.value.naturalWidth;
    imageHeight.value = imageToDetect.value.naturalHeight;
    getDetect();
  };
  imageSrc.value = URL.createObjectURL(e.target.files[0]);
  imageName.value = e.target.files[0].name.split(".").slice(0, -1).join(".");
};

// Run object detection on the current image
const getDetect = async () => {
  const img = imageToDetect.value;
  results.value = await cocoSsdModel.detect(img);
};
</script>

<style scoped lang="scss">

.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh; 
}
.background {
  width: 500px;
  height: 400px;
  position: relative;
}

.display-none {
  display: none;
}

.polaroid {
  transform: rotate(4deg);
  width: 300px;
  background-color: white;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
  margin-bottom: 25px;
  transition: transform 0.2s;
  margin: auto;
  border: 15px solid white;
  border-radius: 5px;
  position: relative;
  margin: 20px;
  z-index: 10;
}
.particle-1 {
  width: 23vmax;
  height: 22vmax;
  position: absolute;
  background-image: linear-gradient(45deg, #3023ae 0%, #ff0099 100%);
  left: -8vmin;
  top: -3vmin;
  animation: morph 15s linear infinite alternate, spin 20s linear infinite;
  will-change: border-radius, transform;
  /* transform-origin: 55% 55%; */
  pointer-events: none;
}
.particle-2 {
  width: 20vmax;
  height: 22vmax;
  position: absolute;
  background-image: linear-gradient(45deg, #3023ae 0%, #ff0099 100%);
  right: 14vmin;
  bottom: -1vmin;
  animation: morph 17s linear infinite alternate-reverse,
    spin 22s linear infinite reverse;
  will-change: border-radius, transform;
  /* transform-origin: 55% 55%; */
  pointer-events: none;
}

@keyframes morph {
  0% {
    border-radius: 40% 60% 60% 40%/70% 30%;
  }

  to {
    border-radius: 40% 60%;
  }
}

@keyframes spin {
  to {
    transform: rotate(2turn);
  }
}

.polaroid:hover {
  transform: scale(1.1);
}

.image-container {
  // position: relative;
  // display: inline-block;
  // max-width: 100%;
  // max-height: 80%;
  width: 300px; /* Ajusta esto al tamaño que necesites */
  height: 300px; /* Ajusta esto al tamaño que necesites */
  border: 1px solid black;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: white;
}
.image-container img {
  width: 100%;
  height: auto;
  min-width: 100px;
  min-height: 100px;
}

.bounding-box {
  position: absolute;
  border: 2px solid;
  box-sizing: border-box;
}

.bounding-box span {
  background-color: white;
  color: black;
  position: absolute;
  bottom: 100%;
  font-size: 14px;
}
.image-caption {
  text-align: center;
  padding: 10px;
}
</style>
