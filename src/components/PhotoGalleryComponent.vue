```vue
<template>
  <ion-card>
    <ion-card-header>
      <ion-card-title>Photo Gallery</ion-card-title>
    </ion-card-header>

    <ion-card-content>

      <!-- NO PHOTOS -->
      <div
        v-if="photos.length === 0"
        class="empty-gallery"
      >
        <p>No photos yet.</p>
      </div>

      <!-- PHOTO GALLERY -->
      <div
        v-else
        class="photo-grid"
      >
        <div
          v-for="(photo, index) in photos"
          :key="index"
          class="photo-item"
        >
          <img
            :src="photo"
            alt="Captured photo"
          />
        </div>
      </div>

    </ion-card-content>
  </ion-card>
</template>

<script setup lang="ts">

import {
  ref,
  onMounted,
  onUnmounted
} from 'vue';

import {
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent
} from '@ionic/vue';

const photos = ref<string[]>([]);

/* =========================
   LOAD SAVED PHOTOS
========================= */
const loadPhotos = () => {

  const savedPhotos =
    localStorage.getItem('photos');

  if (savedPhotos) {

    try {

      photos.value =
        JSON.parse(savedPhotos);

    } catch (error) {

      console.error(
        'Unable to load photos:',
        error
      );

      photos.value = [];
    }

  } else {

    photos.value = [];
  }
};

/* =========================
   WHEN COMPONENT LOADS
========================= */
onMounted(() => {

  // Load existing photos
  loadPhotos();

  // Listen for new photos
  window.addEventListener(
    'photos-updated',
    loadPhotos
  );
});

/* =========================
   WHEN COMPONENT IS REMOVED
========================= */
onUnmounted(() => {

  window.removeEventListener(
    'photos-updated',
    loadPhotos
  );
});

</script>

<style scoped>

.empty-gallery {
  text-align: center;
  padding: 30px 10px;
  color: #777;
}

.photo-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
}

.photo-item img {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 10px;
  display: block;
}

</style>
```
