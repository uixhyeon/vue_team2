<template>
  <div class="check-page">
    <!-- 상단 로고 -->
    <header class="header">
      <img src="/public/images/mains/header/logo-1.png" alt="마타주 로고" class="logo" />
    </header>

    <!-- 중앙 카드 -->
    <main class="check-card">
      <h1>예약 확인</h1>
      <p class="desc">예약번호를 입력해주세요!</p>

      <form @submit.prevent="checkReservation">
        <div class="form-group">
          <input
            type="text"
            placeholder="예: 20251029-1234"
            v-model="reservationCode"
          />
        </div>

        <button type="submit" class="btn primary full">확인하기</button>
      </form>

      <div class="link-row">
        <button type="button" class="link-btn" @click="openFindModal">
          예약번호 찾기
        </button>
        <p>|</p>
        <RouterLink class="link-btn" to="/changereserv">예약 변경하기</RouterLink>
    
      </div>
    </main>

    <!-- 예약번호 찾기 모달 -->
    <FindReservationModal v-if="showModal" @close="showModal = false" />
  </div>
</template>

<script setup>
import { ref, getCurrentInstance } from "vue";
import { RouterLink } from "vue-router";
import FindReservationModal from "@/views/sign/FindReservModal.vue";

// 전역 $alert 접근
const { appContext } = getCurrentInstance();
const $alert = appContext.config.globalProperties.$alert;

const reservationCode = ref("");
const showModal = ref(false);

// 예약번호 확인
const checkReservation = () => {
  if (!reservationCode.value) {
    $alert("예약번호를 입력해주세요.");
    return;
  }

  $alert(`예약번호 ${reservationCode.value} 확인 중...`);
};


// 모달
const openFindModal = () => {
  $alert("예약번호가 맞지 않습니다");
};
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

.check-page {
      padding: 80px 0 100px;
  background: #f5f7f7;
  // min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;

}


.header {
  margin-bottom: 20px;

  .logo {
    width: 120px;
    height: auto;
  }
}


.check-card {
  background: #fff;
  width: 440px;
  padding: 50px 60px;
  border-radius: $radius-m ;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.06);
  text-align: center;

  h1 {
    font-size: 22px;
    font-weight: 700;
    margin-bottom: 10px;
    color: #333;
  }

  .desc {
    font-size: 15px;
    color: #555;
    margin-bottom: 30px;
  }

  .form-group {
    margin-bottom: 25px;

    input {
      width: 100%;
      border: 1px solid #ccc;
      border-radius: $radius-s ;
      padding: 12px 14px;
      font-size: 15px;
      outline: none;

      &:focus {
        border-color: $color_main;
      }
    }
  }

  .btn.primary.full {
    width: 100%;
    padding: 14px 0;
    font-size: 15px;
    font-weight: 600;
    background: $color_main;
    color: #fff;
    border: none;
    border-radius: $radius-s ;
    cursor: pointer;

    &:hover {
      background: $color_main_deep;
    }
  }

  .link-row {
    margin-top: 16px;
    display: flex;
    justify-content: center;
    gap: 8px;
    align-items: center;

    p {
      color: #999;
      margin: 0;
    }

    //버튼링크
    .link-btn {
      background: none;
      border: none;
      color: #666;
      text-decoration: underline;
      font-size: 14px;
      cursor: pointer;
      padding: 0;
     

      &:hover {
        color: $color_main;
      }
    
    }

    .router-link-active {
      font-size: 14px;
      color: #666;
      text-decoration: underline;
      cursor: pointer;

      &:hover {
        color: $color_main;
      }
    }
  }
}
</style>
