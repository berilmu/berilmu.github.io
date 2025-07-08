<script setup>
import { ref } from "vue";
import StrokeAnimation from "../vueWritter/practice/StrokeAnimate.vue";
import { useSoalStore } from "@/stores/soalStore";

const props = defineProps({ question: Object, jawaban: Object, NomorSoal: Number });
const emit = defineEmits(["answered"]);

const writer1 = ref(null);
const soalStore = useSoalStore();
function startAnimationHandler() { writer1.value?.restartAnimation(); }
function handleFinish() {
  soalStore.updateJawabanUserPartial(props.NomorSoal, { finished: true });
  emit("answered", true);
}
</script>
<template>
  <div class="cov">
    <h2 class="text-xl font-bold text">{{ question.soal }}</h2>
    <div class="box">
      <div class="button-top-container"><a class="ms-3" @click="startAnimationHandler"><i class="bi bi-repeat" /></a></div>
      <StrokeAnimation
        class="stroke"
        ref="writer1"
        :strokesData="question.path.strokes"
        :size="'10rem'"
        strokeColor="blue"
        :delayBetweenStrokes="1"
        :animationDuration="5"
        @finished="handleFinish"
      />
    </div>
  </div>
</template>
<style scoped>
.cov{padding-top:5rem}.stroke{flex:3;display:flex;justify-content:center;align-items:center;width:100%}.box{display:flex;flex-direction:column;height:100%;width:100%}.text{font-size:1rem;margin-bottom:3rem}
</style>
