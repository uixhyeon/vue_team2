<template>
  <div class="modal-overlay" @click.self="close">
    <div class="modal-box">
      <header class="modal-header">
        <h2>예약번호 찾기</h2>
        <button class="close-btn" @click="close">&times;</button>
      </header>

      <div class="modal-body">
        <p class="desc">등록 시 사용한 이름과 휴대폰 번호를 입력해주세요.</p>

        <!-- 이름 입력 -->
        <div class="form-group">
          <label>이름</label>
          <input type="text" v-model="name" placeholder="이름 입력" />
        </div>

        <!-- 휴대폰 번호 입력 -->
        <div class="form-group">
          <label>휴대폰 번호</label>
          <input type="text" v-model="phone" placeholder="- 없이 입력" />
        </div>

        <!-- 찾기 버튼 -->
        <button class="btn primary full" @click="findReservation">
          예약번호 찾기
        </button>

        <!-- 결과 -->
        <div v-if="foundCode" class="result">
          <p>예약번호: <strong>{{ foundCode }}</strong></p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, getCurrentInstance } from "vue";
const emit = defineEmits(["close"]);

const name = ref("");
const phone = ref("");
const foundCode = ref("");

//  전역 $alert 접근 (Composition API 방식)
const { appContext } = getCurrentInstance();
const $alert = appContext.config.globalProperties.$alert;

const findReservation = () => {
  if (!name.value || !phone.value) {
    $alert("이름과 휴대폰 번호를 모두 입력해주세요.");
    return;
  }

  // 실제 API 연동 시 여기서 서버 요청
  foundCode.value = "20251029-1234"; // 예시값
  $alert("예약번호를 찾았습니다!");
};

const close = () => {
  emit("close");
};
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;


.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}


.modal-box {
  background: #fff;
  width: 400px;
  border-radius: $radius-m;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  animation: fadeIn 0.25s ease;
  overflow: hidden;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}


.modal-header {
  background: $color_main;
  color: #fff;
  padding: 16px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;

  h2 {
    font-size: 18px;
    margin: 0;
  }

  .close-btn {
    background: none;
    border: none;
    font-size: 22px;
    color: #fff;
    cursor: pointer;
  }
}


.modal-body {
  padding: 24px 28px;

  .desc {
    font-size: 14px;
    color: #555;
    margin-bottom: 18px;
  }

  .form-group {
    margin-bottom: 16px;

    label {
      font-size: 13px;
      font-weight: 600;
      display: block;
      margin-bottom: 5px;
    }

    input {
      width: 100%;
      border: 1px solid #ccc;
      border-radius: $radius-s;
      padding: 10px;
      outline: none;

      &:focus {
        border-color: $color_main;
      }
    }
  }

  .btn.primary.full {
    width: 100%;
    background: $color_main;
    color: #fff;
    padding: 12px 0;
    border-radius: $radius-s;
    border: none;
    cursor: pointer;
    margin-top: 8px;
    font-weight: 600;

    &:hover {
      background: $color_main_deep;
    }
  }

  .result {
    margin-top: 15px;
    background: #f7f7f7;
    border-radius: $radius-s;
    padding: 12px 15px;
    text-align: center;
    color: #333;

    strong {
      color: $color_main_deep;
    }
  }
}
</style>
