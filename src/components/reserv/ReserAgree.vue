<template>
  <transition name="fade">
    <div v-if="show" class="terms-overlay" @click.self="$emit('close')">
      <div class="terms-box">
        <h2>이용약관 동의</h2>
        <div class="terms-list">
          <label class="all-agree">
            <input type="checkbox" v-model="agreeAll" @change="toggleAll" />
            전체 동의합니다
          </label>

          <ul>
            <li>
              <label><input type="checkbox" v-model="agree14" /> [필수] 만 14세 이상입니다.</label>
            </li>
            <li>
              <label><input type="checkbox" v-model="agreeService" /> [필수] 서비스 이용약관 동의 (보기)</label>
            </li>
            <li>
              <label><input type="checkbox" v-model="agreePrivacy" /> [필수] 개인정보 수집 및 이용동의 (보기)</label>
            </li>
            <li>
              <label><input type="checkbox" v-model="agreeThird" /> [필수] 개인정보 제3자 제공동의 (보기)</label>
            </li>
            <li>
              <label><input type="checkbox" v-model="agreeMarketing" /> [선택] 마케팅 목적 개인정보 수집 및 이용동의 (보기)</label>
            </li>
          </ul>
        </div>

        <div class="terms-actions">
          <button class="btn cancel" @click="$emit('close')">취소</button>
          <button class="btn agree" :disabled="!canProceed" @click="handleAgree">
            이용동의
          </button>
        </div>
      </div>
    </div>
  </transition>
</template>

<script setup>
import { ref, computed, watch } from "vue";

// props & emits
const props = defineProps({
  show: Boolean,
});
const emit = defineEmits(["close", "agree"]);

// 체크박스 상태
const agreeAll = ref(false);
const agree14 = ref(false);
const agreeService = ref(false);
const agreePrivacy = ref(false);
const agreeThird = ref(false);
const agreeMarketing = ref(false);

// 전체 선택
const toggleAll = () => {
  const value = agreeAll.value;
  agree14.value = value;
  agreeService.value = value;
  agreePrivacy.value = value;
  agreeThird.value = value;
  agreeMarketing.value = value;
};

// 필수 항목 모두 체크해야 진행 가능
const canProceed = computed(() =>
  agree14.value && agreeService.value && agreePrivacy.value && agreeThird.value
);

// 전체 체크 상태 자동 반영
watch([agree14, agreeService, agreePrivacy, agreeThird, agreeMarketing], () => {
  agreeAll.value =
    agree14.value && agreeService.value && agreePrivacy.value && agreeThird.value && agreeMarketing.value;
});

// "이용동의" 버튼 클릭
const handleAgree = () => {
  if (canProceed.value) {
    emit("agree");
  }
};
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

//이용약관 모달
.terms-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9998;
}

.terms-box {
  background: #fff;
  border-radius: $radius-m;
  width: min(90vw, 480px);
  max-height: 80vh;
  overflow-y: auto;
  padding: 28px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  text-align: left;

  h2 {
    font-size: 1.25rem;
    font-weight: 700;
    color: #333;
    margin-bottom: 1rem;
    text-align: center;
  }

  .terms-list {
    ul {
      list-style: none;
      padding: 0;
      margin: 0.5rem 0 0;
      li {
        margin-bottom: 0.5rem;
        font-size: 0.9rem;
        color: #555;
      }
    }

    input[type="checkbox"] {
      accent-color: $color_main;
      margin-right: 8px;
    }

    .all-agree {
      display: block;
      font-weight: 700;
      margin-bottom: 1rem;
    }
  }

  .terms-actions {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-top: 1.5rem;

    .btn {
      border: none;
      border-radius: $radius-s;
      font-weight: 600;
      padding: 10px 18px;
      cursor: pointer;
      min-width: 100px;
      font-size: 0.95rem;

      &.cancel {
        background: #ddd;
        color: #333;
      }
      &.agree {
        background: $color_main;
        color: #fff;
      }
      &.agree:hover {
        background: $color_main_deep;
      }

      &:disabled {
        opacity: 0.5;
        cursor: not-allowed;
      }
    }
  }
}

//페이드
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
