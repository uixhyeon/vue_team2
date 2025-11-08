<template>
  <div class="join-page">
    <header class="header">
      <div class="logo left">로고</div>
      <h1>회원가입</h1>
      <div class="logo right">로고</div>
    </header>

    <div class="join-card">
      <form @submit.prevent="onSubmit">
        <!-- 휴대폰 입력 -->
        <div class="form-group">
          <label>휴대폰 번호</label>
          <div class="input-row">
            <input
              type="text"
              placeholder="번호를 입력해주세요 (-제외)"
              v-model="phone"
              @input="validatePhone"
            />
            <button
              type="button"
              class="btn small"
              :disabled="!isPhoneValid"
              @click="generateCode"
            >
              인증 요청
            </button>
          </div>
 <p class="desc" v-if="phoneError" :style="{ color: '#e53935' }">
  {{ phoneError }}
</p>
        </div>

        <!-- 인증번호 -->
        <div class="form-group">
          <label>인증 번호</label>
          <div class="input-row">
            <input
              type="text"
              placeholder="인증번호"
              v-model="inputCode"
              :disabled="!sentCode"
            />
            <button
              type="button"
              class="btn small"
              :disabled="!sentCode"
              @click="verifyCode"
            >
              인증 하기
            </button>
          </div>
          <p class="desc" v-if="isVerified" style="color:#53b4a1">
            인증번호가 확인되었습니다.
          </p>
        </div>

        <!-- 약관 -->
        <div class="terms">
          <div class="term-header">
            <label>
              <input
                type="checkbox"
                v-model="allChecked"
                ref="master"
                :aria-checked="allChecked ? 'true' : 'false'"
              />
              전체 동의합니다
            </label>
          </div>
          <ul>
            <li v-for="(term, i) in terms" :key="i">
              <label>
                <input type="checkbox" v-model="term.checked" />
                {{ term.label }}
              </label>
            </li>
          </ul>
        </div>

        <!-- 이용동의 -->
        <button
          type="submit"
          class="btn primary full"
          :disabled="!canSubmit"
        >
          이용동의
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();

import { getCurrentInstance } from "vue";
const { appContext } = getCurrentInstance();


// 전화번호 유효성
const phone = ref("");
const phoneError = ref("");
const isPhoneValid = ref(false);

const validatePhone = () => {
  const pattern = /^010\d{8}$/; // 010으로 시작 + 8자리 = 11자리
  if (phone.value === "") {
    phoneError.value = ""; // 비어있을 때만 표시
    isPhoneValid.value = false;
    return;
  }
  if (!pattern.test(phone.value)) {
    phoneError.value = "숫자 11자리를 입력해주세요 (- 제외)";
    isPhoneValid.value = false;
  } else {
    phoneError.value = ""; // 유효하면 메시지 제거
    isPhoneValid.value = true;
  }
};

//인증번호 발송 확인
const sentCode = ref("");      //만든번호
const inputCode = ref("");     // 유저 입력 번호
const isVerified = ref(false); // 인증 ox

const generateCode = () => {
  // 6자리 랜덤
  const randomCode = Math.floor(Math.random() * 900000) + 100000;
  sentCode.value = randomCode.toString();
  inputCode.value = sentCode.value; // 자동입력
  isVerified.value = false; // 새로 보냈으면 다시 인증
 appContext.config.globalProperties.$alert("인증번호가 발송되었습니다.");

};

const verifyCode = () => {
  if (inputCode.value === sentCode.value && sentCode.value !== "") {
    isVerified.value = true;
  
    appContext.config.globalProperties.$alert("인증번호가 확인되었습니다");

  } else {
    isVerified.value = false;
   
   appContext.config.globalProperties.$alert("인증번호가 일치하지 않습니다");
 
  }
};

// 이용약관
const terms = ref([
  { label: "[필수] 만 14세 이상입니다.", checked: false, required: true },
  { label: "[필수] 서비스 이용약관 동의 (보기)", checked: false, required: true },
  { label: "[필수] 개인정보 수집 및 이용동의 (보기)", checked: false, required: true },
  { label: "[필수] 개인정보 제3자 제공동의 (보기)", checked: false, required: true },
  { label: "[선택] 마케팅 목적 개인정보 수집 및 이용동의 (보기)", checked: false, required: false },
]);

const allChecked = computed({
  get() {
    return terms.value.every((t) => t.checked);
  },
  set(val) {
    terms.value.forEach((t) => (t.checked = val));
  },
});

const requiredAllChecked = computed(() =>
  terms.value.filter((t) => t.required).every((t) => t.checked)
);

const master = ref(null);
watch(
  terms,
  () => {
    if (master.value) master.value.indeterminate = false;
  },
  { deep: true, immediate: true }
);

//인증성공 필수체크해야
const canSubmit = computed(() => isVerified.value && requiredAllChecked.value);

//서브밋
const onSubmit = () => {
  if (!canSubmit.value) {
  
    appContext.config.globalProperties.$alert("휴대폰 인증과 필수약관 동의를 완료해 주세요");

    return;
  }

  
  router.push("/signup2");
  appContext.config.globalProperties.$alert("동의완료");
};
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

//전체구조
.join-page {
  // min-height: 100vh;
  background: #f5f7f7;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  z-index: 0;
  padding-bottom: 100px;
}

//상단
.header {
  height: 200px;
  position: relative;
  width: 100%;
  background: $color_main;
  color: #fff;
  text-align: center;
  padding: 50px 0;
  z-index: 1;

  h1 {
    font-size: 26px;
    font-weight: 700;
    margin: 0;
    position: relative;
    z-index: 2;
  }
// 임시로 지움
  .logo {
    position: absolute;
    top: 50%;
    transform: translateY(-50%) rotate(45deg);
    color: transparent;
    width: 70px;
    height: 70px;
    line-height: 70px;
    font-weight: 600;
    text-align: center;
    border-radius: $radius-s;
    z-index: 2;

    &.left {
      left: 80px;
    }

    &.right {
      right: 80px;
    }
  }
}

//회원가입카드 
.join-card {
  background: #fff;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.06);
  // border-radius: $radius-s;
  padding: 50px 60px;
  width: 500px;
  position: relative;
  z-index: 10;
  margin-top: -60px; //곂쳐지게

  form {
    position: relative;
    z-index: 10;
  }
}

//라인형 입력
.form_group {
  margin-bottom: 25px;

  label {
    font-size: 14px;
    font-weight: 600;
    color: #333;
    display: block;
    margin-bottom: 6px;
  }

  input,
  select {
    width: 100%;
    border: none;
    border-bottom: 1px solid #e7e7e7;
    background: transparent;
    font-size: 14px;
    padding: 10px 4px;
    outline: none;
    color: #333;
    transition: border-color 0.2s ease;

    &:focus {
      border-bottom: 1px solid #53b4a1;
    }
  }

  select {
    appearance: none;
    background: url("data:image/svg+xml;utf8,<svg fill='%2355b4a1' height='10' viewBox='0 0 20 20' width='10'><path d='M5 7l5 5 5-5H5z'/></svg>")
      no-repeat right 10px center;
    background-size: 12px;
    padding-right: 28px;
  }

  .label {
    font-size: 12px;
    color: #888;
    margin-top: 4px;
  }
}

//입력구조 보충
.form-group {
  margin-bottom: 25px;

  label {
    font-size: 14px;
    font-weight: 600;
    color: #333;
    display: block;
    margin-bottom: 6px;
  }

  .input-row {
    display: flex;
    gap: 10px;

    input {
      flex: 1;
      border: none;
      border-bottom: 1px solid #ccc;
      font-size: 14px;
      padding: 8px 4px;
      outline: none;

      &:focus {
        border-bottom-color: $color_main_background;
      }
      
    }
  }

  .desc {
    font-size: 12px;
    color: #999;
    margin-top: 4px;
  }
}

//약관
.terms {
  border-top: 1px solid #e7e7e7;
  padding-top: 15px;
  margin-top: 10px;

  .term-header {
    margin-bottom: 10px;
    label {
      font-weight: 700;
    }
  }

  ul {
    list-style: none;
    padding: 0;
    margin: 0;

    li {
      font-size: 13px;
      color: #555;
      margin-bottom: 6px;
    }
  }

  input[type="checkbox"] {
    margin-right: 8px;
    accent-color: $color_main_light;
  }
}

//버튼
.btn {
  background: $color_main;
  color: #fff;
  border: none;
  border-radius: $radius-s;
  cursor: pointer;
  font-weight: 600;
  padding: 12px 16px;

  &.small {
    padding: 8px 14px;
    font-size: 13px;
  }

  &.primary.full {
    width: 100%;
    padding: 16px 0;
    font-size: 15px;
    margin-top: 25px;
  }

  &:hover {
    background: $color_main_deep ;
  }

      &:disabled {
      background: #ccc;
      color: #fff;
      cursor: not-allowed;
      opacity: 0.8;
    }
}

//반응형
@media (max-width: 600px) {
  .join-card {
    width: 90%;
    padding: 30px 20px;
  }
  .header .logo {
    display: none;
  }
}

//이용동의 활성화 비활성화
.btn.primary.full {
  background: $color_main; // 활성화 기본색
  color: #fff;
  transition: background 0.3s ease;

  &:hover:not(:disabled) {
    background: $color_main_deep; // hover 시 진한색
  }

  &:disabled {
    background: #ccc; // 비활성화 회색
    color: #fff;
    cursor: not-allowed;
    opacity: 0.8;
  }
}

</style>
