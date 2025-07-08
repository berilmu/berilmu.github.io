<script setup>
import { ref, onMounted, onUnmounted, computed } from "vue";
import PracticeStrokeWritter from "../vueWritter/practice/StrokeWritter.vue";
import { useSoalStore } from "@/stores/soalStore";

const soalStore = useSoalStore();
const props = defineProps({ question: Object, NomorSoal: Number, jawaban: Object });
const emit = defineEmits(["answered"]);

const isAnswered = ref(false);
const size = ref(256);

// ✅ Safe strokesData
const strokesData = computed(() => Array.isArray(props.question?.path?.strokes) ? props.question.path.strokes : []);

const hasilLatihan = ref({});
function handleLatihanSelesai(result){
  isAnswered.value=true;
  hasilLatihan.value[props.question.id]=result;
  soalStore.setJawabanUser(props.NomorSoal,result);
  emit("answered",true);
}
function updateSize(){const w=window.innerWidth;size.value=w<200?100:w<210?150:w<260?200:256;}
onMounted(()=>{updateSize();window.addEventListener("resize",updateSize);});
onUnmounted(()=>window.removeEventListener("resize",updateSize));
</script>
<template>
  <div class="cov">
    <h2 class="text-xl font-bold text">{{ question.soal }}</h2>
    <div class="box">
      <PracticeStrokeWritter
        class="stroke"
        :avgsdH="question.path.avgsdH"
        :avgsdL="question.path.avgsdL"
        :size="size"
        :strokesData="strokesData"
        :savedResult="hasilLatihan[question.id]"
        :reviewData="jawaban"
        @finished="handleLatihanSelesai"
      />
    </div>
  </div>
</template>
<style scoped>
.cov{padding-top:3rem}.stroke{flex:3;display:flex;justify-content:center;align-items:center;width:100%}.box{align-items:center;display:flex;flex-direction:column;height:100%;width:100%}.text{font-size:1rem;margin-bottom:3rem}
</style>