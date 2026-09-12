<template>
  <ion-card>
    <ion-card-header>
      <ion-card-title>Photo Gallery</ion-card-title>
    </ion-card-header>

    <ion-card-content>

      <ion-button expand="block" @click="takePhoto">
        <ion-icon slot="start" :icon="cameraIcon"></ion-icon>
        Take Photo
      </ion-button>

      <div v-if="showCamera" class="camera-container">
        <video
          ref="video"
          autoplay
          playsinline
          class="camera-preview"
        ></video>

        <ion-button
          expand="block"
          color="success"
          @click="capturePhoto"
        >
          Capture Photo
        </ion-button>

        <ion-button
          expand="block"
          fill="outline"
          color="danger"
          @click="closeCamera"
        >
          Cancel
        </ion-button>
      </div>

      <canvas ref="canvas" style="display: none"></canvas>

      <ion-text v-if="errorMessage" color="danger">
        <p>{{ errorMessage }}</p>
      </ion-text>

      <ion-grid v-if="photos.length > 0">
        <ion-row>
          <ion-col
            size="6"
            size-md="4"
            v-for="(photo, index) in photos"
            :key="index"
          >
            <ion-img :src="photo"></ion-img>
          </ion-col>
        </ion-row>
      </ion-grid>

      <ion-text v-else-if="!showCamera" color="medium">
        <p>No photos yet.</p>
      </ion-text>

    </ion-card-content>
  </ion-card>
</template>

<script setup lang="ts">
import { ref, onUnmounted } from 'vue';

import {
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonButton,
  IonIcon,
  IonText,
  IonGrid,
  IonRow,
  IonCol,
  IonImg
} from '@ionic/vue';

import { camera as cameraIcon } from 'ionicons/icons';

const photos = ref<string[]>([]);
const errorMessage = ref('');

const video = ref<HTMLVideoElement | null>(null);
const canvas = ref<HTMLCanvasElement | null>(null);

const showCamera = ref(false);

let stream: MediaStream | null = null;

const takePhoto = async () => {
  errorMessage.value = '';

  try {
    if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
      errorMessage.value =
        'Camera is not supported by this browser.';
      return;
    }

    stream = await navigator.mediaDevices.getUserMedia({
      video: {
        facingMode: {
          ideal: 'environment'
        }
      },
      audio: false
    });

    showCamera.value = true;

    await new Promise(resolve => setTimeout(resolve, 100));

    if (video.value) {
      video.value.srcObject = stream;
      await video.value.play();
    }

  } catch (error) {
    console.error('Camera Error:', error);

    errorMessage.value =
      'Unable to access camera. Please allow camera permission.';
  }
};

const capturePhoto = () => {
  if (!video.value || !canvas.value) {
    return;
  }

  const videoElement = video.value;
  const canvasElement = canvas.value;

  canvasElement.width = videoElement.videoWidth;
  canvasElement.height = videoElement.videoHeight;

  const context = canvasElement.getContext('2d');

  if (!context) {
    errorMessage.value = 'Unable to capture photo.';
    return;
  }

  context.drawImage(
    videoElement,
    0,
    0,
    canvasElement.width,
    canvasElement.height
  );

  const photoData = canvasElement.toDataURL(
    'image/jpeg',
    0.9
  );

  photos.value.push(photoData);

  closeCamera();
};

const closeCamera = () => {
  if (stream) {
    stream.getTracks().forEach(track => {
      track.stop();
    });

    stream = null;
  }

  if (video.value) {
    video.value.srcObject = null;
  }

  showCamera.value = false;
};

onUnmounted(() => {
  closeCamera();
});
</script>

<style scoped>
.camera-container {
  margin-top: 20px;
}

.camera-preview {
  width: 100%;
  max-height: 500px;
  object-fit: cover;
  border-radius: 12px;
  background: black;
  display: block;
  margin-bottom: 15px;
}
</style>