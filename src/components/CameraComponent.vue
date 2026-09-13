<template>
  <ion-card>
    <ion-card-header>
      <ion-card-title>Camera</ion-card-title>
    </ion-card-header>

    <ion-card-content>
      <ion-button expand="block" @click="takePhoto">
        <ion-icon slot="start" :icon="cameraIcon"></ion-icon>
        Take Photo
      </ion-button>

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

      <ion-text v-else color="medium">
        <p>No photos yet.</p>
      </ion-text>
    </ion-card-content>
  </ion-card>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

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

onMounted(() => {
  const savedPhotos = localStorage.getItem('photos');

  if (savedPhotos) {
    try {
      photos.value = JSON.parse(savedPhotos);
    } catch {
      photos.value = [];
    }
  }
});

const savePhotos = () => {
  localStorage.setItem(
    'photos',
    JSON.stringify(photos.value)
  );

  window.dispatchEvent(
    new Event('photos-updated')
  );
};

const takePhoto = async () => {
  errorMessage.value = '';

  console.log('TAKE PHOTO BUTTON CLICKED');

  try {
    console.log(
      'Is native:',
      Capacitor.isNativePlatform()
    );

    if (Capacitor.isNativePlatform()) {
      console.log('Checking camera permission...');

      const permission =
        await Camera.checkPermissions();

      console.log(
        'Camera permission:',
        permission.camera
      );

      if (permission.camera !== 'granted') {
        console.log(
          'Requesting camera permission...'
        );

        const requested =
          await Camera.requestPermissions();

        console.log(
          'Requested permission:',
          requested.camera
        );

        if (requested.camera !== 'granted') {
          errorMessage.value =
            'Camera permission was denied.';
          return;
        }
      }
    }

    console.log('Opening camera...');

    const image = await Camera.getPhoto({
      quality: 90,
      allowEditing: false,
      resultType: CameraResultType.DataUrl,
      source: CameraSource.Camera
    });

    console.log('Camera returned:', image);

    if (image.dataUrl) {
      photos.value.push(image.dataUrl);
      savePhotos();
    }

  } catch (error) {
    console.error(
      'CAMERA ERROR:',
      error
    );

    errorMessage.value =
      'Unable to access camera. Please allow camera permission.';
  }
};
</script>