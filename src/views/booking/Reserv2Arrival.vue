<template>
  <div class="form_card" :class="{ open: isOpen }">
    <!-- ✅ 헤더 클릭 시 열기/닫기 -->
    <div class="card_header" @click="$emit('toggle')">
      <h3>짐 가져오기</h3>

      <!-- ✅ 모든 입력 완료 시 초록색 체크 표시 -->
      <i
        v-if="isComplete"
        class="fa-solid fa-check"
        style="color:#4CAF50; font-size:1.1rem;"
      ></i>
    </div>

    <!-- ✅ 내용 -->
    <transition name="fade">
      <div v-show="isOpen" class="card_content" @click.stop>
        <!-- 픽업 주소 -->
        <div class="form_group">
          <label>픽업 주소*</label>
          <div class="addr-input">
            <input
              type="text"
              placeholder="픽업하실 주소를 입력해주세요"
              v-model="localForm.pickupAddress"
              readonly
              @focus="$emit('touch', 'pickupAddress')"
            />
            <button type="button" class="mini-btn" @click="$emit('openPickup')">
              주소 검색
               <i class="fa-solid fa-magnifying-glass"></i>
            </button>
          </div>
          <p v-if="touched.pickupAddress && errors.pickupAddress" class="error">
            {{ errors.pickupAddress }}
          </p>
        </div>

        <!-- 상세주소 -->
        <div class="form_group">
          <label>상세주소*</label>
          <input
            type="text"
            placeholder="상세주소를 입력해주세요"
            v-model="localForm.pickupAddressDetail"
            @focus="$emit('touch', 'pickupAddressDetail')"
          />
          <p
            v-if="touched.pickupAddressDetail && errors.pickupAddressDetail"
            class="error"
          >
            {{ errors.pickupAddressDetail }}
          </p>
        </div>

        <!-- 픽업일 -->
    <div class="form_group">
  <label>픽업일*</label>
<VueDatePicker
  v-model="localForm.pickupDate"
  locale="ko"
  :enable-time-picker="false"
  :auto-apply="false"
  format="yyyy-MM-dd"
  class="date-picker"
  placeholder="날짜를 선택해 주세요"
  :action-row="{ showSelect: true, selectText: '선택 완료', showCancel: false }"
  @update:model-value="$emit('touch', 'pickupDate')"
  @focus-input="$emit('touch', 'pickupDate')"
/>


  <p v-if="touched.pickupDate && errors.pickupDate" class="error">
    {{ errors.pickupDate }}
  </p>
</div>

        <!-- @@@@@@@@@@@@@@@@추가함 -->
       <div class="btn-grup-wrap">

         <div class="btn-group">
           <!-- <p style="padding-left: 3px; margin-bottom:14px;"> 추가서비스를 이용하시겠어요?</p> -->
           <button type="button" class="card-btn left" @click="$emit('move', 'locker' )">
             사물함 예약을 수정해요
            </button>
            <button   type="button"
            class="card-btn right"
            @click="$emit('move', 'luggage' )">
            짐을 집으로 받아요
          </button>
        </div>
      </div>


      </div>
    </transition>
  </div>
</template>

<script setup>
import { computed } from "vue";
import VueDatePicker from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

const props = defineProps({
  form: { type: Object, required: true },
  isOpen: { type: Boolean, default: true },
  errors: { type: Object, default: () => ({}) },
  touched: { type: Object, default: () => ({}) }, // ✅ 추가
});

const emit = defineEmits([
  "update:form",
  "openPickup",
  "toggle",
  "touch",
  "move" // ✅ 단일 이동 이벤트
]);


const localForm = computed({
  get: () => props.form,
  set: (val) => emit("update:form", val),
});

// ✅ 모든 입력 완료 시 체크 아이콘 표시
const isComplete = computed(() => {
  const f = props.form;
  return f.pickupAddress && f.pickupAddressDetail && f.pickupDate;
});
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

.form_card {
  background: #fff;
  border-radius: $radius-m ;
  border: 1px solid #f0f0f0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  position: relative;
  padding: 15px 40px 10px;
  transition: all 0.25s ease;
  color: #444;
  font-size: $text-sm;
  box-sizing: border-box;

  &::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 12px;
    background: $color_main;
    border-top-left-radius: $radius-m ;
    border-top-right-radius: $radius-m ;
  }

  &.open {
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.08);
  }

  .card_header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
    margin: 15px 0;

    h3 {
      font-size: $text-md;
      font-weight: 600;
      color: #333;
      margin: 0;
    }

    i {
      font-size: 1rem;
      color: $color_main;
    }
  }

  .form_group {
    margin-bottom: 20px;

    label {
      display: block;
      font-size: $label-md;
      font-weight: 500;
      color: #555;
      margin-bottom: 6px;
      padding-left: 3px;
    }

    input,
    select {
      width: 100%;
      border: none;
      border-bottom: 1px solid #e7e7e7;
      background: transparent;
      font-size: $label-md;
      color: #333;
      padding: 10px;
      transition: border-color 0.2s ease;

      &:focus {
        border-bottom: 1px solid $color_main_light;
        outline: none;
      }

      &::placeholder {
        color: #aaa;
      }
    }
  }

  .addr-input {
    display: flex;
    gap: 8px;
    align-items: center;

    .mini-btn {
      width: 120px;
      padding: 8px 12px;
      border-radius: $radius-s;
      background: $color_main;
      color: #fff;
      border: none;
      cursor: pointer;
      font-size: $label-sm;
      transition: 0.2s;

      &:hover {
        background: $color_main_deep;
      }
    }
  }
}

.error {
  color: #e53935;
  font-size: 0.85rem;
  margin-top: 4px;
  padding-left: 3px;
  line-height: 1.4;
}


// //모바일 버튼
// ==============================
.btn-group {
  display: flex;
  justify-content: space-between;
  gap: 14px;
  margin-top: 14px;
   padding-bottom: 25px;

  .card-btn {
    flex: 1;
    border-radius: $radius-m;
    padding: 20px 0;
    font-size: 1rem;
    font-weight: 600;
    text-align: center;
    border: none;
    cursor: pointer;
    transition: all 0.25s ease;
  }

//왼쪽
  .card-btn.left {
    background: #f5f5f5;
    color: #616161;
    border: 1.5px solid #e0e0e0;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.03);

    &:hover {
      background: #eaeaea;
      // transform: translateY(-2px);
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.08);
    }
  }
    
   //오른쪽
  .card-btn.right {
    background: rgba(83, 180, 161, 0.15); //15%
    color: #2E7E73;                     
    border: 1.5px solid rgba(83, 180, 161, 0.25);
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
  
    &:hover {
      background: rgba(83, 180, 161, 0.25);
      // transform: translateY(-2px);
      box-shadow: 0 4px 10px rgba(83, 180, 161, 0.15);
    }
  }
  }


  

  @media (min-width: 1025px) {
  .btn-group {
    display: none !important;
  }
}
//모바일
@media (max-width: 480px) {
  .btn-group {
    flex-direction: column;
    gap: 10px;

    .card-btn {
      padding: 16px 0;
      font-size: 0.95rem;
    }
  }
}


//버튼그룹 피씨에서는 안보임
@media (min-width: 1025px) {
  .btn-group-wrap {
    display: none !important;
  }
}
// ============뷰데이픽커================

// ============뷰데이픽커================
//==========외부================
.date-picker {
  width: 100%; /* ✅ 입력라인은 카드 전체 폭 기준으로 복구 */
  display: block;
  position: relative;

  /* 내부 인풋 래퍼 초기화 */
  :deep(.dp__input_wrap),
  :deep(.dp__main) {
    width: 100%;
    background: transparent !important;
    border: none !important;
    box-shadow: none !important;
  }

  /* 인풋 필드 */
  :deep(.dp__input) {
    width: 100% !important; /* ✅ 카드 전체 폭 */
    background: transparent !important;
    border: none !important;
    border-bottom: 1px solid #e7e7e7 !important;
    border-radius: 0 !important;
    padding: 8px 8px !important;
    font-size: clamp(0.85rem, 0.9vw, 0.95rem) !important;
    color: #333 !important;
    transition: border-color 0.25s ease;

    &::placeholder {
      color: #777 !important;
    }

    &:focus {
      border-bottom: 1px solid $color_main_light !important;
      outline: none !important;
    }
  }

  /* 달력 아이콘 제거 */
  :deep(.dp__input_icon) {
    display: none !important;
  }
}

//==========팝업================
/* =========================================================
  📅 VueDatePicker 팝업 스타일 (입력창 아래 자연스럽게 뜨는 형태)
========================================================= */

/* 팝업 전체 래퍼 */
:deep(.dp__outer_menu_wrap) {
  position: absolute !important;          /* ✅ 인풋 기준 */
  top: 100% !important;                   /* ✅ 바로 밑 */
  left: 50% !important;                   /* ✅ 가운데 정렬 */
  transform: translateX(-50%) !important; /* ✅ 좌우 보정 */
  margin-top: 8px !important;

  z-index: 9999 !important;
  width: 350px !important;                /* ✅ 가로 350px 고정 */
  height: auto !important;                /* ✅ 세로 자동 */
  max-width: calc(100vw - 40px) !important; /* ✅ 모바일 대응 */
  max-height: 90vh !important;            /* ✅ 너무 커지면 스크롤 */
  overflow-y: auto !important;

  border-radius: $radius-m  !important;
  background: rgba(255, 255, 255, 0.98) !important;
  border: 1px solid #d2e8e8 !important;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12) !important;
  padding: 20px !important;
}

/* ✅ 팝업 배경 딤 제거 (인풋 아래 뜨는 형태라 필요 없음) */
:deep(.dp__outer_menu_wrap::before) {
  display: none !important;
}

/* ✅ 뾰족한 삼각형 제거 */
:deep(.dp__arrow_top),
:deep(.dp__arrow_bottom) {
  display: none !important;
}

/* =========================================================
  내부 달력 스타일
========================================================= */
:deep(.dp__menu_inner) {
  padding: 16px !important;
  background: #fff !important;
  border-radius: $radius-m  !important;
}

:deep(.dp__calendar_header) {
  display: flex !important;
  justify-content: space-between !important;
  align-items: center !important;
  font-weight: 600 !important;
  color: #333 !important;
  margin-bottom: 10px !important;
}

:deep(.dp__calendar_item) {
  font-size: 0.9rem !important;
  border-radius: $radius-s  !important;
  padding: 6px 0 !important;
  transition: 0.2s;
}

/* ✅ 선택된 구간 색상 */
:deep(.dp__range_start),
:deep(.dp__range_end) {
  background: $color_main !important;
  color: #fff !important;
  border-radius: $radius-s  !important;
}

:deep(.dp__range_between) {
  background: #eaf8f6 !important;
}

/* =========================================================
  🎨 테마 컬러 변수 (색상 전체 통일)
========================================================= */
:deep(.dp__theme_light),
:deep(.dp__theme_dark) {
  --dp-primary-color: #53b4a1 !important;
  --dp-primary-text-color: #fff !important;
  --dp-hover-color: #449b8a !important;
  --dp-hover-text-color: #fff !important;

  --dp-success-color: #53b4a1 !important;
  --dp-success-text-color: #fff !important;

  --dp-icon-color: #53b4a1 !important;
  --dp-hover-icon-color: #3a8c88 !important;

  --dp-secondary-color: #f7fcfb !important;
  --dp-border-color: #d2e8e8 !important;
  --dp-menu-border-color: #d2e8e8 !important;
  --dp-range-between-dates-background-color: #eaf8f6 !important;

  /* ✅ “선택 완료” 버튼 컬러 */
  --dp-action-button-bg: #53b4a1 !important;
  --dp-action-button-hover-bg: #449b8a !important;
  --dp-action-button-text-color: #fff !important;
}

/* =========================================================
  [2] 버튼 영역 커스터마이징
========================================================= */
:deep(.dp__action_row) {
  display: flex !important;
  justify-content: center !important;

}

/* ❌ 취소 버튼 숨기기 */
:deep(.dp__cancel) {
  display: none !important;
}

/* ✅ “선택 완료” 버튼 */
:deep(.dp__select) {
  flex: 1 !important;
  width: 100% !important;
  padding: 14px 0 !important;
  font-size: 1rem !important;
  font-weight: 600 !important;
  color: #fff !important;
  background-color: #53b4a1 !important;
  border-radius: $radius-s  !important;
  border: none !important;
  cursor: pointer !important;
  transition: background-color 0.25s ease !important;
}
:deep(.dp__select:hover) {
  background-color: #449b8a !important;
}
// 버튼이 커지게




/* 데스크탑에서 팝업을 화면 중앙(카드 기준 아닌 뷰포트 중앙)으로 고정 */

  :deep(.dp__outer_menu_wrap) {
    position: fixed !important;
    top: 50% !important;
    left: 50% !important;
    transform: translate(-50%, -50%) !important;
    z-index: 99999 !important;
    width: 560px !important;
    max-width: 350px !important;
    max-height: 80vh !important;
    margin-top: 0 !important; /* 기존 margin-top 방지 */
  }

// ...existing code...
:deep(.dp__action_select),
:deep(.dp__select),
:deep(.custom-select) {
  display: block !important;
  width: 100% !important;
  // padding: 16px 0 !important;
  background: #3E9C9B !important;
  color: #fff !important;
  text-align: center !important;
  font-weight: 700 !important;
  font-size: 1rem !important;
  border-radius: $radius-m !important;
  border: none !important;
  cursor: pointer !important;
  transition: background 0.25s ease !important;
}
:deep(.dp__action_select:hover),
:deep(.dp__select:hover),
:deep(.custom-select:hover) {
  background: darken(#3E9C9B, 6%) !important;
}

:deep(.dp__select) {
  display: block !important;
  width: 100% !important;
  padding: 16px 0 !important;
  background: #3E9C9B !important;
  color: #fff !important;
  text-align: center !important;
  font-weight: 700 !important;
  font-size: 1rem !important;
  border-radius: $radius-m !important;
  border: none !important;
  cursor: pointer !important;
  transition: background 0.25s ease !important;
}
:deep(.dp__select:hover) {
  background: darken(#3E9C9B, 6%) !important;
}

</style>
