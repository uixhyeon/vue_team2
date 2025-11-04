<template>
  <!-- 모달 배경 -->
  <div class="modal-overlay" @click.self="closeModal">
    <!-- 모달 박스 -->
    <div class="modal-box">
      <header class="modal-header">
        <h2>아이디 찾기</h2>
        <button class="close-btn" @click="closeModal">&times;</button>
      </header>

      <div class="modal-body">
        <p class="desc">가입 시 등록한 휴대폰 번호를 입력해주세요.</p>

        <!-- 휴대폰 번호 입력 -->
        <div class="form-group">
          <label>휴대폰 번호</label>
          <div class="input-row">
            <input type="text" placeholder="- 없이 입력" v-model="phone" />
            <button type="button" class="btn small" @click="sendCode">인증 요청</button>
          </div>
        </div>

        <!-- 인증번호 입력 -->
        <div v-if="codeSent" class="form-group">
          <label>인증번호</label>
          <div class="input-row">
            <input type="text" placeholder="인증번호 입력" v-model="verifyCode" />
            <button type="button" class="btn small" @click="verify">확인</button>
          </div>
        </div>

        <!-- 결과 표시 -->
        <div v-if="foundId" class="result">
          <p>회원님의 아이디는 <strong>{{ foundId }}</strong> 입니다.</p>
        </div>
      </div>

      <footer class="modal-footer">
        <button class="btn primary full" @click="closeModal">닫기</button>
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, getCurrentInstance } from "vue";
const emit = defineEmits(["close"]);

const phone = ref("");
const verifyCode = ref("");
const codeSent = ref(false);
const foundId = ref("");

// 전역 $alert 접근 (Composition API에서)
const { appContext } = getCurrentInstance();
const $alert = appContext.config.globalProperties.$alert;

// 인증 요청
const sendCode = () => {
  if (!phone.value) {
    $alert("휴대폰 번호를 입력해주세요.");
    return;
  }
  $alert("인증번호를 발송했습니다.");
  codeSent.value = true;
};

// 인증 확인
const verify = () => {
  if (verifyCode.value === "1234") {
    foundId.value = "uixhyeon123"; // 예시
    $alert("인증이 완료되었습니다.");
  } else {
    $alert("인증번호가 올바르지 않습니다.");
  }
};

// 모달 닫기
const closeModal = () => {
  emit("close");
};
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

//모달배경
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}

//박스
.modal-box {
  background: #fff;
  width: 420px;
  border-radius: $radius-m;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  overflow: hidden;
  animation: fadeIn 0.25s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-10px); }
  to { opacity: 1; transform: translateY(0); }
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
    font-size: 24px;
    color: #fff;
    cursor: pointer;
  }
}

//바디
.modal-body {
  padding: 24px 28px;

  .desc {
    font-size: 14px;
    color: #666;
    margin-bottom: 20px;
  }

  .form-group {
    margin-bottom: 18px;

    label {
      font-size: 14px;
      font-weight: 600;
      color: #333;
      display: block;
      margin-bottom: 6px;
    }

    .input-row {
      display: flex;
      gap: 8px;

      input {
        flex: 1;
        border: 1px solid #ccc;
        border-radius: $radius-s ;
        padding: 10px;
        outline: none;

        &:focus {
          border-color: $color_main;
        }
      }
    }
  }

  .result {
    margin-top: 15px;
    background: #f7f7f7;
    border-radius: $radius-s ;
    padding: 12px 15px;
    text-align: center;

    strong {
      color: $color_main_deep;
      font-weight: 700;
    }
  }
}

.modal-footer {
  padding: 20px;
  border-top: 1px solid #eee;
  text-align: center;
}


.btn {
  background: $color_main;
  color: #fff;
  border: none;
  border-radius: $radius-s ;
  cursor: pointer;
  font-weight: 600;
  padding: 10px 14px;

  &.small {
    padding: 8px 12px;
    font-size: 13px;
  }

  &.primary.full {
    width: 100%;
    padding: 14px 0;
    font-size: 15px;
  }

  &:hover {
    background: $color_main_deep;
  }
}
</style>
