<template>
  <div class="join-page">
    <header class="header">
      <div class="logo left">
        <img src="/images/mains/header/logo-1.png" alt="로고" />
      </div>
      <h1>회원가입</h1>
      <div class="logo right">
        <img src="/images/mains/header/logo-1.png" alt="로고" />
      </div>
    </header>

    <div class="join-card">
      <form @submit="submitForm">
        <!-- 이메일 -->
        <div class="form_group">
          <label>이메일 아이디*</label>
          <div class="email-row">
            <input
              type="text"
              placeholder="이메일 아이디"
              v-model="emailId"
              @blur="validateEmail"
            />
            <span>@</span>
            <select v-model="emailDomain" @change="handleDomainChange">
              <option disabled value="">도메인 선택</option>
              <option value="gmail.com">gmail.com</option>
              <option value="naver.com">naver.com</option>
              <option value="daum.net">daum.net</option>
              <option value="kakao.com">kakao.com</option>
              <option value="custom">직접 입력</option>
            </select>
          </div>
          <input
            v-if="emailDomain === 'custom'"
            type="text"
            placeholder="직접 입력"
            v-model="customDomain"
            @blur="validateEmail"
            class="custom-domain"
          />
          <p class="label" :style="{ color: errors.email ? '#e53935' : '#888' }">
            {{ errors.email || '' }}
          </p>
        </div>

        <!-- 비밀번호 -->
        <div class="form_group">
          <label>비밀번호*</label>
          <input
            type="password"
            placeholder="영문 숫자 포함 8글자 이상"
            v-model="form.password"
            @blur="validatePassword"
          />
          <p class="label" :style="{ color: errors.password ? '#e53935' : '#888' }">
            {{ errors.password || '' }}
          </p>
        </div>

        <!-- 비밀번호 확인 -->
        <div class="form_group">
          <label>비밀번호 확인*</label>
          <input
            type="password"
            placeholder="비밀번호를 다시 입력해주세요"
            v-model="form.confirm"
            @blur="validateConfirm"
          />
          <p class="label" :style="{ color: errors.confirm ? '#e53935' : '#888' }">
            {{ errors.confirm || '' }}
          </p>
        </div>

        <!-- 선택 입력 정보 -->
        <div class="title-wrap"><h2>선택입력 정보</h2></div>

        <div class="form_group">
          <label>성함</label>
          <input
            type="text"
            placeholder="성함을 입력해주세요"
            v-model="form.name"
            @blur="validateName"
          />
          <p class="label" :style="{ color: errors.name ? '#e53935' : '#888' }">
            {{ errors.name || '' }}
          </p>
        </div>

       <!-- 주소 (카카오 주소검색 연결) -->
<div class="form_group">
  <label>주소</label>

  <div class="address-row">
    <input
      type="text"
      placeholder="지번 및 도로명 주소를 입력해주세요"
      v-model="form.address"
      readonly
      @blur="validateAddress"
    />
    <button type="button" class="btn search" @click="searchAddress">
      <span>주소 검색</span>
      <i class="fa-solid fa-magnifying-glass"></i>
    </button>
  </div>

  <p class="label" :style="{ color: errors.address ? '#e53935' : '#888' }">
    {{ errors.address || '' }}
  </p>

  <div class="gapp"></div>

  <input style="padding-top:20px ;"
    type="text"
    placeholder="상세주소를 입력해주세요"
    v-model="form.detail"
  />
</div>


<!-- 제출 버튼 -->
<button type="submit" class="btn primary full" :disabled="!isFormValid">
  입력 완료
</button>

      </form>
    </div>
  </div>
</template>
<script setup>
import { ref, computed, onMounted, getCurrentInstance } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const { appContext } = getCurrentInstance();

//이메일 
const emailId = ref("");
const emailDomain = ref("");
const customDomain = ref("");
const fullEmail = computed(() => {
  if (!emailId.value) return "";
  if (emailDomain.value === "custom") return `${emailId.value}@${customDomain.value}`;
  return `${emailId.value}@${emailDomain.value}`;
});

const handleDomainChange = () => {
  if (emailDomain.value !== "custom") customDomain.value = "";
};

//입력들
const form = ref({
  password: "",
  confirm: "",
  name: "",
  address: "",
  detail: "",
});

const errors = ref({
  email: "",
  password: "",
  confirm: "",
  name: "",
  address: "",
});

//카카오 주소검색모달으로 
onMounted(() => {
  // ✅ 카카오 주소검색 스크립트가 없을 경우 로드
  if (!window.daum || !window.daum.Postcode) {
    const script = document.createElement("script");
    script.src = "//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js";
    document.body.appendChild(script);
  }
});

function searchAddress() {
  if (!window.daum || !window.daum.Postcode) {
    appContext.config.globalProperties.$alert("주소검색 모듈이 아직 로드되지 않았습니다. 잠시 후 다시 시도해주세요 ⏳");
    return;
  }

  new window.daum.Postcode({
    oncomplete: (data) => {
      form.value.address = data.roadAddress || data.jibunAddress;
      appContext.config.globalProperties.$alert("주소가 입력되었습니다");
    },
  }).open();
}

//유효성 검사
const validateEmail = () => {
  const pattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  errors.value.email = pattern.test(fullEmail.value)
    ? ""
    : "올바른 이메일이 아닙니다.";
};

const validatePassword = () => {
  const pattern = /^[A-Za-z\d]{8,}$/;
  errors.value.password = pattern.test(form.value.password)
    ? ""
    : "영문 또는 숫자를 8자 이상 입력해주세요.";
  validateConfirm();
};

const validateConfirm = () => {
  errors.value.confirm =
    form.value.password === form.value.confirm
      ? ""
      : "비밀번호가 일치하지 않습니다.";
};

const validateName = () => {
  const len = form.value.name.trim().length;
  errors.value.name =
    len >= 2 && len <= 10 ? "" : "이름은 2~10자로 입력해주세요.";
};

const validateAddress = () => {
  errors.value.address = form.value.address ? "" : "주소를 입력해주세요.";
};

//유효성
const isFormValid = computed(() =>
  Object.values(errors.value).every((v) => v === "") &&
  fullEmail.value &&
  form.value.password &&
  form.value.confirm &&
  form.value.name &&
  form.value.address
);

//제출
const submitForm = (e) => {
  e.preventDefault();
  validateEmail();
  validatePassword();
  validateConfirm();
  validateName();
  validateAddress();

  if (!isFormValid.value) {
    appContext.config.globalProperties.$alert("입력 정보를 다시 확인해주세요");
    return;
  }

  appContext.config.globalProperties.$alert(`회원가입 완료!\n${fullEmail.value} 🎉`);
  router.push("/login");
};
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;
.join-page {
  // min-height: 100vh;
  background: #f5f7f7;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0 0 100px;
  position: relative;
  z-index: 1;
}


.header {
  position: relative;
  height: 180px;
  width: 100%;
  background: #59b5b3;
  text-align: center;
  padding: 60px 0;

  h1 {
    color: #fff;
    font-size: 28px;
    font-weight: 700;
    position: relative;
    z-index: 2;
  }

  .logo {
    position: absolute;
    top: 50%;
    transform: translateY(-50%) rotate(45deg);
    width: 70px;
    height: 70px;
    opacity: 0.1;
     display: none;

    img {
      width: 100%;
      height: 100%;
      opacity: 0.12;
      display: none;
    }

    &.left {
      left: 80px;
    }

    &.right {
      right: 80px;
    }
  }
}


.join-card {
  background: #fff;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.06);
  padding: 50px 60px;
  width: 500px;
  position: relative;
  z-index: 3; 
  margin-top: -60px;  


 
}

//이메일
.email-row {
  display: flex;
  align-items: center;
  gap: 8px;
  input {
    flex: 1;
    border: none;
    border-bottom: 1px solid #ccc;
    padding: 8px 4px;
    font-size: 14px;
    outline: none;
    &:focus {
      border-color: $color_main_light;
    }
  }
  span {
    font-size: 15px;
    color: #333;
  }
  select {
    flex: 1;
    border: none;
    border-bottom: 1px solid #ccc;
    font-size: 14px;
    padding: 8px 4px;
    background: transparent;
    outline: none;
    &:focus {
      border-color: $color_main_light;
    }
  }
}

.custom-domain {
  width: 100%;
  margin-top: 6px;
  border: none;
  border-bottom: 1px solid #ccc;
  font-size: 14px;
  padding: 8px 4px;
  outline: none;
  &:focus {
    border-color: $color_main_light;
  }
}

//공통
.form_group {
  margin-bottom: 25px;
  label {
    font-size: 14px;
    font-weight: 600;
    color: #333;
    margin-bottom: 6px;
    display: block;
  }
  input {
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
      border-bottom: 1px solid $color_main_light;
    }
  }
  .label {
    font-size: 12px;
    color: #888;
    margin-top: 4px;
  }
}


.title-wrap {
  display: flex;
  flex-direction: column;
  gap: 2px;
  margin: 30px 0 15px;
  padding-bottom: 4px;
  

  h2 {
    padding-top: 10px;
    font-size: 17px;
    font-weight: 700;
    color: #333;
  }
}


.btn {
  background: $color_main;
  color: #fff;
  border: none;
  border-radius: $radius-s;
  cursor: pointer;
  font-weight: 600;
  padding: 12px 16px;
  transition: 0.3s;
  &.primary.full {
    width: 100%;
    padding: 16px 0;
    font-size: 15px;
    margin-top: 25px;
  }
  &:hover:not(:disabled) {
    background: $color_main_deep;
  }
  &:disabled {
    background: #ccc;
    cursor: not-allowed;
    opacity: 0.8;
  }
}

// 버튼추가
.address-row {
  display: flex;
  align-items: center;
  gap: 10px;

  input {
    flex: 1;
    border: none;
    border-bottom: 1px solid #e7e7e7;
    background: transparent;
    font-size: 14px;
    padding: 10px 4px;
    outline: none;
    color: #333;

    &:focus {
      border-bottom: 1px solid $color_main_light;
    }
  }

  .btn.line {
    background: #f3f3f3;
    color: #333;
    border-radius: $radius-s;
    font-size: 13px;
    font-weight: 600;
    padding: 8px 14px;
    border: none;
    cursor: pointer;
    transition: all 0.2s ease;

    &:hover {
      background: #eaeaea;
    }
  }
}
// 돋보기 추가
.address-row {
  display: flex;
  align-items: center;
  gap: 10px;

  input {
    flex: 1;
    border: none;
    border-bottom: 1px solid #e7e7e7;
    background: transparent;
    font-size: 14px;
    padding: 10px 4px;
    outline: none;
    color: #333;

    &:focus {
      border-bottom: 1px solid $color_main_light;
    }
  }

  .btn.search {
    display: flex;
    align-items: center;
    gap: 6px;
    background: $color_main;
    color: #fff;
    border-radius: $radius-s; /* ✅ 라운드형 */
    font-size: 13px;
    font-weight: 600;
    padding: 9px 16px;
    border: none;
    cursor: pointer;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.08);
    transition: all 0.25s ease;

    i {
      font-size: 14px;
    }

    &:hover {
      background: $color_main_deep;
      // transform: translateY(-1px)
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.12);
    }
  }
}

//===================추가함======

.join-page {
  
  background: #f5f7f7;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  z-index: 0;
 
}

.header {
  position: relative;
  width: 100%;
  height: 200px;
  background: $color_main;
  color: #fff;
  text-align: center;
  padding: 60px 0;
  z-index: 1;

  h1 {
    font-size: 28px;
    font-weight: 700;
    position: relative;
    z-index: 3;
    margin: 0;
  }

  .logo {
    position: absolute;
    top: 50%;
    transform: translateY(-50%) rotate(45deg);
    background: rgba(255, 255, 255, 0.3);
    width: 70px;
    height: 70px;
    text-align: center;
    line-height: 70px;
    font-weight: 600;
    border-radius: $radius-s;
    color: #333;
    z-index: 2;
    opacity: 0.15;

    &.left {
      left: 80px;
    }

    &.right {
      right: 80px;
    }
  }
}


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
      background: transparent;
      color: #333;
      transition: border-color 0.2s ease;

      &:focus {
        border-bottom: 1px solid $color_main_light;
      }
    }
  }

  .desc {
    font-size: 12px;
    color: #999;
    margin-top: 4px;
  }
}

.terms {
  border-top: 1px solid #e7e7e7;
  padding-top: 15px;
  margin-top: 20px;

  .term-header {
    margin-bottom: 10px;

    label {
      font-weight: 700;
      font-size: 14px;
      color: #222;
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
      line-height: 1.4;
    }
  }

  input[type="checkbox"] {
    accent-color: $color_main_light;
    margin-right: 8px;
  }
}

.btn {
  background: $color_main;
  color: #fff;
  border: none;
  // border-radius: $radius-s;
  cursor: pointer;
  font-weight: 600;
  padding: 12px 16px;
  transition: background 0.3s ease;

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

  &:hover:not(:disabled) {
    background: $color_main_deep;
  }

  &:disabled {
    background: #ccc;
    cursor: not-allowed;
    opacity: 0.8;
  }
}

@media (max-width: 600px) {
  .join-card {
    width: 90%;
    padding: 30px 20px;
    // margin-top: 30px;
  }

  .header {
    height: 160px;
    padding: 40px 0;

    .logo {
      display: none; 
    }

    h1 {
      font-size: 22px;
    }
  }
}


</style>
