```vue
<template>
  <ion-card>
    <ion-card-header>
      <ion-card-title>Photo Gallery</ion-card-title>
    </ion-card-header>

    <ion-card-content>

      <!-- TAKE PHOTO BUTTON -->
      <ion-button expand="block" @click="takePhoto">
        <ion-icon slot="start" :icon="cameraIcon"></ion-icon>
        Take Photo
      </ion-button>

      <!-- ERROR MESSAGE -->
      <ion-text v-if="errorMessage" color="danger">
        <p>{{ errorMessage }}</p>
      </ion-text>

      <!-- PHOTO GALLERY -->
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

      <!-- NO PHOTOS -->
      <ion-text v-else color="medium">
        <p>No photos yet.</p>
      </ion-text>

    </ion-card-content>
  </ion-card>
</template>

<script setup lang="ts">
import { ref } from 'vue';

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

import {
  Camera,
  CameraResultType,
  CameraSource
} from '@capacitor/camera';

import { Capacitor } from '@capacitor/core';

const photos = ref<string[]>([]);
const errorMessage = ref('');

const takePhoto = async () => {
  errorMessage.value = '';

  try {

    // =========================
    // ANDROID / IOS
    // =========================
    if (Capacitor.isNativePlatform()) {

      const image = await Camera.getPhoto({
        quality: 90,
        allowEditing: false,
        resultType: CameraResultType.DataUrl,
        source: CameraSource.Camera
      });

      if (image.dataUrl) {
        photos.value.push(image.dataUrl);
      }

      return;
    }

    // =========================
    // BROWSER / IONIC SERVE
    // =========================
    if (
      !navigator.mediaDevices ||
      !navigator.mediaDevices.getUserMedia
    ) {
      errorMessage.value =
        'Camera is not supported by this browser.';
      return;
    }

    const stream = await navigator.mediaDevices.getUserMedia({
      video: {
        facingMode: {
          ideal: 'environment'
        }
      },
      audio: false
    });

    const video = document.createElement('video');

    video.srcObject = stream;
    video.autoplay = true;
    video.playsInline = true;

    await video.play();

    const canvas = document.createElement('canvas');

    await new Promise<void>((resolve) => {
      video.onloadedmetadata = () => {
        resolve();
      };
    });

    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;

    const context = canvas.getContext('2d');

    if (!context) {
      stream.getTracks().forEach(track => track.stop());

      errorMessage.value =
        'Unable to capture photo.';

      return;
    }

    context.drawImage(
      video,
      0,
      0,
      canvas.width,
      canvas.height
    );

    const photoData = canvas.toDataURL(
      'image/jpeg',
      0.9
    );

    photos.value.push(photoData);

    // Stop camera
    stream.getTracks().forEach(track => track.stop());

  } catch (error) {

    console.error('Camera Error:', error);

    errorMessage.value =
      'Unable to access camera. Please allow camera permission.';
  }
};
</script>
```
