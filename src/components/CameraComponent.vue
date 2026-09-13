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
  IonText
} from '@ionic/vue';

import { camera as cameraIcon } from 'ionicons/icons';

import {
  Camera,
  CameraResultType,
  CameraSource
} from '@capacitor/camera';

import { Capacitor } from '@capacitor/core';

const errorMessage = ref('');

const savePhoto = (photo: string) => {
  const savedPhotos = localStorage.getItem('photos');
  let photos: string[] = [];

  if (savedPhotos) {
    try {
      photos = JSON.parse(savedPhotos);
    } catch {
      photos = [];
    }
  }

  photos.push(photo);
  localStorage.setItem(
    'photos',
    JSON.stringify(photos)
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
        savePhoto(image.dataUrl);
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