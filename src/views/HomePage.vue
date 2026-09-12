<template>
  <IonPage>
    <IonContent :fullscreen="true">

      <div class="page-container">

        <div class="app-container">

          <!-- App Title -->
          <div class="app-title">
            <span>IONIC</span>
            <h1>Photo Gallery</h1>
            <p>Capture and save your favorite moments</p>
          </div>

          <!-- Camera Component -->
          <CameraComponent />

          <!-- Photo Gallery Component -->
          <PhotoGalleryComponent />

        </div>

      </div>

    </IonContent>
  </IonPage>
</template>

<script setup lang="ts">

import {
  IonPage,
  IonContent
} from '@ionic/vue';

import CameraComponent from '@/components/CameraComponent.vue';
// @ts-expect-error Vue SFC module is resolved at build time.
import PhotoGalleryComponent from '../components/PhotoGalleryComponent.vue';
</script>

<style scoped>

ion-content {
  --background: #090b10;
}

.page-container {
  width: 100%;
  min-height: 100%;
  display: flex;
  justify-content: center;
  box-sizing: border-box;
  padding:
    calc(24px + env(safe-area-inset-top))
    16px
    calc(24px + env(safe-area-inset-bottom));
}

.app-container {
  width: 100%;
  max-width: 500px;
}

.app-title {
  margin-bottom: 24px;
}

.app-title span {
  color: #3880ff;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 3px;
}

.app-title h1 {
  margin: 4px 0 6px;
  color: white;
  font-size: 2rem;
  font-weight: 600;
}

.app-title p {
  margin: 0;
  color: #8b93a3;
  font-size: 0.95rem;
}

</style>