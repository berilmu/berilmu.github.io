<template>
  <div>
    <!-- Header -->
    <Header :judul="dataHeader" class="mb-4" />
    <div class="container py-4">

      <!-- Timeline -->
      <ul class="timeline list-unstyled">
        <li
          v-for="(item, index) in items"
          :key="item.id"
          class="timeline-item d-flex flex-column gap-2 mb-4 p-3 border rounded shadow-sm"
          :class="{ 'timeline-done': item.done }"
        >
          <!-- Bar Indicator + Icon -->
          <div class="d-flex align-items-center">
            <div class="me-3 flex-shrink-0">
              <div
                class="timeline-icon d-flex align-items-center justify-content-center rounded-circle"
                :class="item.done ? 'bg-success text-white' : 'bg-light border text-dark'"
              >
                <i
                  :class="
                    item.type === 'video'
                      ? 'bi bi-play-circle'
                      : 'bi bi-question-circle'
                  "
                ></i>
              </div>
            </div>

            <div class="flex-grow-1">
              <h6 class="mb-1 fw-semibold">
                {{ index + 1 }}. {{ item.title }}
              </h6>
              <div class="d-flex justify-content-between">
                <small class="text-muted">
                  {{ item.type === 'video' ? `${item.duration} min` : 'Soal' }}
                </small>
                <small v-if="item.done" class="text-success d-flex align-items-center">
                  <i class="bi bi-check-circle-fill me-1"></i> Selesai
                </small>
              </div>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
// Header
import Header from "../components/levelDetailComponents/Header.vue";
import { useRoute } from "vue-router";

const route = useRoute();
const judulGlobal = route.query.judulGlobal || '';
const judulItem = route.query.judulItem || '';

const dataHeader = {
  judul: judulGlobal,
  deskripsi: judulItem,
};

// Dummy list — tambahkan `done: true` jika sudah selesai
const items = [
  { id: 1, title: "01 Pengantar", type: "video", duration: 1, done: true },
  { id: 2, title: "Soal - 1", type: "quiz", done: false },
  { id: 3, title: "02 Apa itu Matematika?", type: "video", duration: 3, done: true },
  { id: 4, title: "Soal - 2", type: "quiz", done: false },
  { id: 5, title: "Soal - 3", type: "quiz", done: false },
  { id: 6, title: "Matematika Lanjutan", type: "video", duration: 4, done: true },
];
</script>

<style scoped>
/* Timeline icon base */
.timeline-icon {
  width: 48px;
  height: 48px;
  font-size: 1.2rem;
}

/* Highlight untuk yang selesai */
.timeline-done {
  background-color: #f1fdf4;
  border-left: 4px solid #198754;
}
</style>
