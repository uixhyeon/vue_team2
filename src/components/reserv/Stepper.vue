<template>
  <div class="stepper">
    <ol>
      <li
        v-for="(step, i) in steps"
        :key="i"
        class="step"
        :class="{ active: currentStep === i + 1 }"
        v-show="windowWidth > 768 || currentStep === i + 1"
      >
        <div class="circle">{{ i + 1 }}</div>
        <p>{{ step }}</p>
        <div v-if="i < steps.length - 1 && windowWidth > 768" class="line"></div>
      </li>
    </ol>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue";

defineProps({
  currentStep: {
    type: Number,
    default: 1,
  },
});

const steps = ["예약하기", "확인 및 결제", "예약 완료"];

const windowWidth = ref(window.innerWidth);
const handleResize = () => (windowWidth.value = window.innerWidth);

onMounted(() => window.addEventListener("resize", handleResize));
onUnmounted(() => window.removeEventListener("resize", handleResize));
</script>

<style lang="scss" scoped>
@use "/src/assets/style/variables" as *;

.stepper {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 1rem 0;

  ol {
    display: flex;
    align-items: center;
    list-style: none;
    gap: 4rem;
    padding: 0;
    margin: 0;
  }
}

.step {
  position: relative;
  text-align: center;
  color: #888;

  /*  원 */
  .circle {
    width: 30px;
    height: 30px;
    border-radius: 50%;
    background: #999;
    color: #fff;
    font-size: 0.8rem;
    font-weight: 700;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0 auto 0.4rem;
    transition: 0.3s;
  }

  p {
    font-size: 0.9rem;
    font-weight: 500;
    color: #888;
    transition: 0.3s;
  }

  &.active {
    .circle {
      background: $color-main;
      color: #fff;
    }
    p {
      color: #222;
      font-weight: 700;
    }
  }

  //점선
  .line {
    position: absolute;
    top: 15px; //중앙
    left: 100%;
    width: 80px;
    height: 1px;
    background: repeating-linear-gradient(
      to right,
      #999,
      #999 6px,
      transparent 6px,
      transparent 12px
    );
  }

  &:last-child .line {
    display: none;
  }
}

//모바일시 점선 논
@media (max-width: 768px) {
  .stepper ol {
    justify-content: center;
    gap: 0;
  }

  .step {
    display: none;
  }

  .step.active {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .line {
    display: none !important;
  }
}
</style>
