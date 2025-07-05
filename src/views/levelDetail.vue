// File: src/views/DetailLevel.vue
<script setup>
import Header from "../components/levelDetailComponents/Header.vue";
import Hero from "../components/levelDetailComponents/HeroComponent.vue";
import ProgresBar from "../components/levelDetailComponents/ProgressBar.vue";
import card from "../components/levelDetailComponents/CardDetailLevel.vue";
import BottomNav from "../components/navComponents/BottomNav.vue";

import { computed, onMounted } from "vue";
import { useRoute } from "vue-router";
const route = useRoute();
const mapel = route.params.mapel;
const slug = route.params.slugmateri;


// Ambil data lesson dari store
import { loadLessonsFromSupabase } from "../service/fetchDummyLesson";
import { useLessonStore } from "@/stores/LessonStore";
const storeLesson = useLessonStore();
// Reactive: ambil data dari store berdasarkan mapel dan slug
const data = computed(() => storeLesson.getLesson(mapel, slug));
console.log("data", data.value);

// Jika data tidak ditemukan
const isDataKosong = computed(() => !data.value);
if (isDataKosong.value) {
  // isi data lesson
  loadLessonsFromSupabase();
}
const dataJudul = computed(() => data.value?.judul || "");

import { useFetchMaterialLessons } from "@/stores/compossable/useFetchMaterialLessons";
import { useMaterialLessonStore } from "@/stores/materialLessonStore";
const { fetchLessons, loading, error } = useFetchMaterialLessons();
const storeMaterialLesson = useMaterialLessonStore();
const lessonId = route.params.id;

onMounted(() => {
  fetchLessons(lessonId);
});
</script>

<template>
  <div>
    <div class="sticky-header">
      <Header :judul="data" />
    </div>
    <div class="container" style="padding-top: 0.5rem">
      <Hero :data="data" />
    </div>
    <div class="container mt-1 pt-1">
      <ProgresBar :progres="90" />
    </div>
    <div class="container space">
      <div v-if="loading">Loading...</div>
      <div v-if="error">Error: {{ error }}</div>
      <div v-else>
        <router-link
          v-for="item in storeMaterialLesson.getAll"
          :key="item.id"
          class="list-group-item list-group-item-action"
          :to="{
            path: `/${mapel}/${item.lastgroup ? 'show' : 'd'}/${item.id}`,
            query: {
              needData: JSON.stringify(item.needData),
              judulGlobal: dataJudul,
              judulItem: item.name,
            },
          }"
        >
          <card
            :id="item.id"
            :deskripsi="item.description"
            :judul="item.name"
            :gambar="item.logo_picture"
            :progres="item.presentase"
            :lastgroup="item.lastgroup"
          />
        </router-link>
      </div>
    </div>
    <BottomNav class="bottomnav" :home="mapel" />
  </div>
</template>

<style scoped>
.sticky-header {
  position: sticky;
  top: 0;
  z-index: 1000;
  background-color: white;
}
@media (min-width: 768px) {
  .bottomnav {
    display: none;
  }
}
@media (max-width: 768px) {
  .space {
    padding-bottom: 4.2rem;
  }
}
</style>
