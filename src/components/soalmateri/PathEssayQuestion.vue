<template>
  <div class="cov">
    <div class="uncov">
      <h2 class="text-xl font-bold mb-2 text">{{ question.soal }}</h2>

      <div class="box">
        <StrokeSvg
          class="stroke"
          :strokesData="question.path.strokes"
          viewBox="24"
        />
      </div>

      <div class="mt-4">
        <input
          v-model="userAnswer"
          type="text"
          placeholder="Jawaban kamu"
          class="border px-2 py-1 rounded w-full mb-2"
          :disabled="isAnswered"
        />

        <button
          @click="submitAnswer"
          class="bg-green-500 text-white px-4 py-2 rounded"
          :disabled="isAnswered || userAnswer.trim() === ''"
        >
          Submit
        </button>
      </div>

      <div v-if="showResult" class="mt-4">
        <p :class="isCorrect ? 'text-success' : 'text-danger'">
          {{
            "Jawaban " +
            (isCorrect
              ? "Benar!"
              : "Salah. Jawaban yang benar adalah " + question.jawaban)
          }}
        </p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from "vue";
import StrokeSvg from "../vueWritter/StrokeSvg.vue";
import { useSoalStore } from "@/stores/soalStore";

const soalStore = useSoalStore();

const props = defineProps({
  question: Object,
  jawaban: Object,
  NomorSoal: Number,
});

const emit = defineEmits(["answered"]);

const userAnswer = ref("");
const showResult = ref(false);
const isCorrect = ref(false);
const isAnswered = ref(false);

function isJawabanValid(jawaban) {
  return (
    jawaban &&
    typeof jawaban.answer === "string" &&
    jawaban.answer.trim().length > 0
  );
}

function resetState() {
  const jawaban = props.jawaban;

  if (isJawabanValid(jawaban)) {
    isAnswered.value = true;
    showResult.value = true;
    isCorrect.value = jawaban.hasil;
    userAnswer.value = jawaban.answer;
    emit("answered", true); // Hanya emit kalau memang sudah dijawab valid
  } else {
    isAnswered.value = false;
    showResult.value = false;
    isCorrect.value = false;
    userAnswer.value = "";
    emit("answered", false); // Kasih tahu parent bahwa soal belum dijawab
  }
}

watch(
  () => props.NomorSoal,
  () => {
    resetState();
  }
);

onMounted(() => {
  resetState();
});

function submitAnswer() {
  const correct =
    userAnswer.value.trim().toLowerCase() ===
    props.question.jawaban.toLowerCase();

  isCorrect.value = correct;
  isAnswered.value = true;
  showResult.value = true;

  soalStore.setJawabanUser(props.NomorSoal, {
    hasil: isCorrect.value,
    answer: userAnswer.value,
  });

  emit("answered", true);
}
</script>

<style scoped>
button:focus {
  outline: none;
}
.stroke {
  flex: 3;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}
.box {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
}
.cov {
  justify-items: center;
}
.uncov {
  padding-top: 4rem;
}
.text {
  font-size: 1rem;
  margin-bottom: 3rem;
}
</style>
