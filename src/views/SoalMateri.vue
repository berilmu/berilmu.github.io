<script setup>
import { useRoute, useRouter } from "vue-router";
import { computed, ref, onMounted } from "vue";

// COMPONENTS
import Header from "../components/soalmateri/header/Header.vue";
import PathEssayQuestion from "../components/soalmateri/PathEssayQuestion.vue";
import AngkaStrokeWriter from "../components/soalmateri/AngkaStrokeWriter.vue";
import NextButton from "../components/soalmateri/NextButton.vue";
import StrokePlayAngka from "@/components/soalmateri/StrokePlayAngka.vue";

// SERVICES
import getSpecialMathData from "../service/soal/getSpecialMathData";
import { transformSoalData, transformJawabanData } from "../service/soalMathFunct";
import { getBundleUserData, simpanSoal } from "@/service/supabase/soalMatematika";

// STORES
import { useSoalStore } from "@/stores/soalStore";
import { useAuthStore } from "@/stores/auth";

const soalStore = useSoalStore();
const route = useRoute();
const router = useRouter();

const path = computed(() => route.path.split("/").pop() || "");

function goBack() {
  soalStore.setIndex(0);
  router.go(-1);
}

const dataquery = JSON.parse(route.query.needData || "null");
const idSoal = route.params.id;

const jumlahSoal = ref(0);
const dataSoal = ref([]);
const dataJawaban = ref([]);
const currentSoal = ref({});
const currentSoalIndex = ref(0);
const isAnswered = ref(false);
const currenAnswer = ref([]);

onMounted(async () => {
  console.log(soalStore.getIsFinish, "isFinish");
  
  const auth = useAuthStore();
  await auth.fetchUserAndProfile();
  const userId = auth.user?.id;

  if (!soalStore.IsAdaSoal()) {
    const soAlFrmDatabase = await getBundleUserData(userId, idSoal);

    if (soAlFrmDatabase.length > 0) {
      dataSoal.value = transformSoalData(soAlFrmDatabase);
      dataJawaban.value = transformJawabanData(soAlFrmDatabase);
      soalStore.setId(idSoal);
    } else if (dataquery != null) {
      soalStore.setId(idSoal);
      dataSoal.value = getSpecialMathData(dataquery);
      const ok = await simpanSoal(dataSoal.value, userId, idSoal);
      if (!ok) return;
    }

    soalStore.tambahBanyakSoal(dataSoal.value);
    dataJawaban.value.forEach((j) => soalStore.setJawabanUser(j.nomor, j));

    jumlahSoal.value = soalStore.jumlahSoalUnik;
    currentSoal.value = soalStore.getSoalByNomor(1)[0];
    currenAnswer.value = soalStore.getJawabanUser(1);
  } else {
    if (idSoal == soalStore.getId) {
      currentSoalIndex.value = soalStore.getIndex;
    }
    jumlahSoal.value = soalStore.jumlahSoalUnik;
    currentSoal.value = soalStore.getSoalByNomor(currentSoalIndex.value + 1)[0];
    currenAnswer.value = soalStore.getJawabanUser(currentSoalIndex.value + 1);
  }
});

function nextSoal() {
  if (currentSoalIndex.value < jumlahSoal.value - 1) {
    currentSoalIndex.value++;
    currenAnswer.value = soalStore.getJawabanUser(currentSoalIndex.value + 1);
    currentSoal.value = soalStore.getSoalByNomor(currentSoalIndex.value + 1)[0];
    soalStore.setIndex(currentSoalIndex.value);
    isAnswered.value = false;
  }
}

function prevSoal() {
  if (currentSoalIndex.value > 0) {
    currentSoalIndex.value--;
    currenAnswer.value = soalStore.getJawabanUser(currentSoalIndex.value + 1);
    currentSoal.value = soalStore.getSoalByNomor(currentSoalIndex.value + 1)[0];
    soalStore.setIndex(currentSoalIndex.value);
    isAnswered.value = true;
  }
}

function handleAnswered(status) {
  isAnswered.value = status;
}
</script>

<template>
  <div>
    <div style="height: 15%"><Header /></div>

    <div v-if="jumlahSoal > 0" style="height: 85%">
      <PathEssayQuestion
        v-if="currentSoal.tipe === 'esay-singkat-stroke'"
        :question="currentSoal"
        :-nomor-soal="currentSoalIndex + 1"
        :jawaban="currenAnswer"
        @answered="handleAnswered"
      />

      <AngkaStrokeWriter
        v-if="currentSoal.tipe === 'menulis-stroke-angka'"
        :question="currentSoal"
        :-nomor-soal="currentSoalIndex + 1"
        :jawaban="currenAnswer"
        @answered="handleAnswered"
      />

      <StrokePlayAngka
        v-if="currentSoal.tipe === 'play-stroke-angka'"
        :question="currentSoal"
        :-nomor-soal="currentSoalIndex + 1"
        :jawaban="currenAnswer"
        @answered="handleAnswered"
      />
    </div>

    <!-- Navigasi setelah selesai (review mode) -->
    <div v-if="soalStore.getIsFinish" class="container mt-4 px-3">
      <div class="d-flex justify-content-between gap-2 flex-wrap">
        <button v-if="currentSoalIndex > 0"
          class="btn btn-outline-secondary rounded-pill flex-fill py-3 fs-6 shadow-sm"
          :disabled="currentSoalIndex === 0"
          @click="prevSoal"
        >
          ⬅️ Sebelumnya
        </button>

        <button v-if="currentSoalIndex < jumlahSoal - 1"
          class="btn btn-outline-secondary rounded-pill flex-fill py-3 fs-6 shadow-sm"
          :disabled="currentSoalIndex === jumlahSoal - 1"
          @click="nextSoal"
        >
          Berikutnya ➡️
        </button>
      </div>
    </div>

    <!-- Navigasi pengerjaan soal -->
    <div v-else>
      <NextButton
        text="Next"
        v-if="isAnswered && currentSoalIndex < jumlahSoal - 1"
        @click="nextSoal"
      />
      <NextButton
        text="Finish"
        v-else-if="isAnswered && currentSoalIndex === jumlahSoal - 1"
        @click="() => soalStore.setIsFinish(true)"
      />
    </div>
  </div>
</template>

<style scoped></style>
