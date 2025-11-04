<template>
  <!-- 전체 프레임 -->
  <div class="page-frame">
    <div class="change-wrap">
      <!-- 상단 영역 -->
      <header class="header">
        <img src="/images/reservation/Asset2.png" alt="마타주 로고" class="logo" />
        <div class="scr">
          <h1>예약 변경</h1>
          <p class="sub">변경할 예약번호를 입력해주세요.</p>
        </div>
      </header>

      <!-- 본문 -->
      <main class="content">
        <!-- 예약번호 입력 -->
        <section class="input-section">
          <label for="res-code">예약번호</label>
          <div class="input-row">
            <input
              id="res-code"
              type="text"
              placeholder="예: 20251029-1234"
              v-model="reservationCode"
            />
            <button class="btn small" @click="loadReservation">조회</button>
          </div>
        </section>

        <!-- 조회 결과 -->
        <section v-if="reservationData" class="info-card">
          <h2>예약 정보</h2>
          <ul>
            <li><strong>이름:</strong> {{ reservationData.name }}</li>
            <li><strong>대여장소:</strong> {{ reservationData.branch }}</li>
            <li><strong>예약일:</strong> {{ reservationData.date }}</li>
            <li><strong>보관물:</strong> {{ reservationData.item }}</li>
          </ul>

          <button class="btn primary full" @click="goToEdit">예약 수정하기</button>
        </section>

        <!-- 버튼 기반 안내 -->
        <div class="hint">
          <p>예약번호를 모르시나요?</p>
          <button type="button" class="link-btn" @click="openFindModal">
            예약번호 찾기
          </button>
        </div>
      </main>
    </div>
  </div>

  <!-- 예약번호 찾기 모달 (최상위 레벨로 이동) -->
  <Teleport to="body">
    <FindReservationModal 
      v-if="showModal" 
      @close="closeModal"
      class="reservation-modal" 
    />
  </Teleport>
</template>


<script setup>
import { ref, getCurrentInstance } from "vue";
import FindReservationModal from "@/views/sign/FindReservModal.vue";  // 경로 수정

// 전역 $alert 접근
const { appContext } = getCurrentInstance();
const $alert = appContext.config.globalProperties.$alert;

const reservationCode = ref("");
const reservationData = ref(null);
const showModal = ref(false);

// 예약번호 조회
const loadReservation = () => {
  if (!reservationCode.value) {
    $alert("예약번호를 입력해주세요.");
    return;
  }

  // 예시 데이터
  reservationData.value = {
    name: "정현영",
    branch: "칠성시장점",
    date: "2025-10-30",
    item: "중형 캐리어",
  };
};

//  예약 수정 이동
const goToEdit = () => {
  $alert("예약 수정 페이지로 이동합니다!");
  // router.push(`/reservation/edit/${reservationCode.value}`);
};

//  모달 제어 함수들
const openFindModal = () => {
  showModal.value = true;
  console.log('모달 열림:', showModal.value);
};

const closeModal = () => {
  showModal.value = false;
};
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

// 전체 구조
.page-frame {
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid $color_main;
  border-radius: $radius-l;
  margin: 40px;
  background: #f6f8f8;
  padding: 40px 0;
  box-sizing: border-box;
}

.change-wrap {
  background: #f6f8f8;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 60px 20px;
  width: 100%;
  max-width: 640px;
}
// 상단영역
.header {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  flex-wrap: wrap;
  gap: 20px;
  width: 100%;
  max-width: 640px;
  margin-bottom: 40px;
  text-align: left;

  .logo {
    width: 220px;
    max-width: 45%;
    height: auto;
    flex-shrink: 0;
  }

  .scr {
    flex: 1 1 200px;
    word-break: keep-all;
  }

  h1 {
    font-size: 26px;
    color: #222;
    font-weight: 700;
    margin-bottom: 6px;
  }

  .sub {
    font-size: 15px;
    color: #555;
  }
}

// 본문
.content {
  background: #fff;
  width: 100%;
  max-width: 640px;
  padding: 40px 50px;
  border-radius: $radius-m;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.input-section {
  margin-bottom: 30px;

  label {
    font-size: 14px;
    font-weight: 600;
    color: #333;
    display: block;
    margin-bottom: 8px;
  }

  .input-row {
    display: flex;
    gap: 10px;

    input {
      flex: 1;
      border: 1px solid #ccc;
      border-radius: $radius-s ;
      padding: 12px;
      font-size: 15px;
      outline: none;

      &:focus {
        border-color: $color_main;
      }
    }

    .btn.small {
      padding: 12px 16px;
      font-size: 14px;
      border: none;
      background: $color_main;
      color: #fff;
      border-radius: $radius-s ;
      cursor: pointer;

      &:hover {
        background: $color_main_deep;
      }
    }
  }
}
//조회결과 카드
.info-card {
  background: #f9fbfb;
  border: 1px solid #e5e8e8;
  border-radius: $radius-s ;
  padding: 24px 28px;
  margin-bottom: 20px;

  h2 {
    font-size: 18px;
    font-weight: 700;
    margin-bottom: 16px;
  }

  ul {
    list-style: none;
    padding: 0;
    margin: 0 0 20px;

    li {
      font-size: 15px;
      color: #444;
      margin-bottom: 8px;

      strong {
        color: #222;
        font-weight: 600;
        margin-right: 8px;
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
}

//하단 안내
.hint {
  margin-top: 16px;
  display: flex;
  justify-content: center;
  gap: 8px;
  align-items: center;
  
  p {
    margin-bottom: 6px;
  }

  .link-btn {
    background: none;
    border: none;
    color: $color_main;
    font-weight: 600;
    text-decoration: underline;
    cursor: pointer;
    font-size: 14px;
    padding: 0;

    &:hover {
      color: $color_main_deep;
    }
  }
}

// 반응형
@media (max-width: 768px) {
  .page-frame {
    border-width: 6px;
    border-radius: $radius-m;
    margin: 20px;
    padding: 20px 0;
  }

  .change-wrap {
    padding: 40px 16px;
  }

  .header {
    flex-direction: column;
    align-items: center;
    text-align: center;
    gap: 12px;

    .logo {
      width: 160px;
      margin-bottom: 8px;
    }

    h1 {
      font-size: 22px;
      line-height: 1.3;
    }

    .sub {
      font-size: 14px;
      line-height: 1.4;
    }
  }

  .content {
    padding: 30px 24px;
  }
}


/* 모달 관련 스타일 추가 */
:deep(.reservation-modal) {
  z-index: 9999;
  position: fixed;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  
  .modal-content {
    background: white;
    padding: 2rem;
    border-radius: $radius-m;
    box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
    position: relative;
    z-index: 10000;
  }

  &::before {
    content: '';
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.5);
    z-index: 9999;
  }
}

</style>
