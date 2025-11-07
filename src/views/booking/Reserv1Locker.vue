<template>
  <div class="form_card" :class="{ open: isOpen }">
    <!--  카드 헤더 클릭으로 열기/닫기 -->
    <div class="card_header" @click="$emit('toggle')">
      <h3>사물함 예약*</h3>

      <!--  모든 입력 완료 시 체크 아이콘 표시 -->
      <i
        v-if="isComplete"
        class="fa-solid fa-check"
        style="color:#4CAF50; font-size:1.1rem;"
      ></i>
    </div>

    <!-- 내용 (토글로 열고닫기) -->
    <transition name="fade">
      <div v-show="isOpen" class="card_content" @click.stop>
        <!-- 이름 -->
       <div class="form_group">
  <label>성함*</label>
  <input
    type="text"
    placeholder="성함을 입력해주세요"
    v-model="localForm.name"
     @focus="$emit('touch', 'name')"
  />
  <p v-if="touched.name && errors.name" class="error">{{ errors.name }}</p>
</div>

<div class="form_group">
  <label>휴대폰 번호*</label>
  <input
    type="text"
    placeholder="01012345678 (-제외)"
    v-model="localForm.phone"
      @focus="$emit('touch', 'phone')"
  />
 
  <p v-if="touched.phone && errors.phone" class="error">{{ errors.phone }}</p>
</div>

<div class="form_group">
  <label>사물함 사이즈*</label>
  <select v-model="localForm.size"
   @focus="$emit('touch', 'size')"
  >
    <option value="">사이즈를 선택해 주세요</option>
    <option>XS</option>
    <option>S</option>
    <option>M</option>
    <option>L</option>
    <option>XL</option>
  </select>

  <p v-if="touched.size && errors.size" class="error">{{ errors.size }}</p>
</div>

<div class="form_group">
  <label>대여 지점*</label>
  <div class="addr-input">
    <input
      type="text"
      placeholder="지점 선택"
      v-model="localForm.address"
      readonly
        @focus="$emit('touch', 'address')"
    />
 <!-- 수정된 버튼 -->
<button
  type="button"
  class="mini-btn"
  :class="{ disabled: !localForm.size }"
  @click="handleOpenBranch"
>
  지점 선택
   <i class="fa-solid fa-magnifying-glass"></i>
</button>

  </div>
 
  <p v-if="touched.address && errors.address" class="error">{{ errors.address }}</p>
</div>

<div class="form_group">
  <label>예약 기간*</label>
  <!-- <VueDatePicker
  v-model="localForm.dateRange"
  range
  locale="ko"
  :enable-time-picker="false"
  format="yyyy-MM-dd"
  placeholder="기간을 선택하세요"
  teleport="#app" 
  /> -->
<VueDatePicker
  v-model="localForm.dateRange"
  range
  locale="ko"
  :enable-time-picker="false"
  format="yyyy-MM-dd"
  placeholder="기간을 선택하세요"
  class="date-picker"
  @open="$emit('touch', 'dateRange')"
  @update:model-value="$emit('touch', 'dateRange')"
>
  <!-- v11에서는 slot 이름이 action-row -->
  <template #action-row="{ selectDate }">
    <button
      class="dp__select custom-select"
      type="button"
      @click="selectDate"
    >
      선택 완료
    </button>
  </template>
</VueDatePicker>


  <p v-if="touched.dateRange && errors.dateRange" class="error">{{ errors.dateRange }}</p>
</div>
<!-- 버튼 부분 수정 -->

<div class="btn-grup-wrap">
  <div class="btn-group">
    <!-- <p style="padding-left: 3px; margin-bottom:14px;"> 배송서비스를 이용하시겠어요?</p> -->
    <button
    type="button"
    class="card-btn left"
    @click="$emit('move', 'arrival')"
    >
    짐을 미리 보내요
  </button>
  
  <button
  type="button"
  class="card-btn right"
  @click="$emit('move', 'luggage')"
  >
  짐을 집으로 받아요
  </button>
</div>
</div>


      </div>
    </transition>
  </div>
</template>
<script setup>
import { computed, getCurrentInstance } from "vue";
import VueDatePicker from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

const props = defineProps({
  form: { type: Object, required: true },
  isOpen: { type: Boolean, default: true },
  errors: { type: Object, default: () => ({}) },
  touched: { type: Object, default: () => ({}) },
   selectedBranch: { type: Object, default: null },
});

const emit = defineEmits(["update:form", "openBranch", "toggle", "touch", "move",
  "update:selectedBranch",]);


const localForm = computed({
  get: () => props.form,
  set: (val) => emit("update:form", val),
});

// 전역 Alert 접근
const { appContext } = getCurrentInstance();

// 버튼 클릭 시 동작
function handleOpenBranch() {
  if (!localForm.value.size) {
    appContext.config.globalProperties.$alert("사물함 사이즈를 먼저 선택해주세요.");
    return;
  }
  emit("openBranch"); // 부모로 전달
}

// ✅ BranchSelectModal에서 선택된 지점 처리
function handleBranchSelect(branch) {
  // 부모로 선택된 지점 전달
  emit("update:selectedBranch", branch);

  // 선택된 지점을 form.address에도 반영 (선택사항)
  localForm.value.address = branch.name;

  // 모달 닫기용 이벤트 (Reservation이 처리)
  emit("openBranch", false);
}

// 입력 완료 여부 (체크 아이콘 표시)
const isComplete = computed(() => {
  const f = props.form;
  return (
    f.name &&
    f.phone &&
    f.size &&
    f.address &&
    f.dateRange &&
    f.dateRange[0] &&
    f.dateRange[1]
  );
});

// <script setup> 내부 하단에 추가
// <script setup> 내부 하단에 추가
function goNextStep() {
  if (window.innerWidth > 1024) return; // PC에서는 작동 안 함
  if (!isComplete.value) {
    appContext.config.globalProperties.$alert("사물함 예약 정보를 모두 입력해주세요.");
    return;
  }
  emit("next"); // 부모에 openSection 변경 요청
}

import { onMounted } from "vue";

onMounted(() => {
  if (!localForm.value.phone) localForm.value.phone = "010";
});


</script>
<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

//기본
.form_card {
  background: #fff;
  border-radius: $radius-m;
  border: 1px solid #f0f0f0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  position: relative;
  padding: 15px 40px 10px;
  transition: all 0.25s ease;
  color: #444;
  font-size: $text-sm;
  box-sizing: border-box;

  // 뷰데이픽커를위한설정
 position: relative; //기준점..부여
  z-index: 1; //위에올리기
  overflow: visible !important; //팝업잘림방지

  &::before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 12px;
    background: $color_main;
    border-top-left-radius: $radius-m;
    border-top-right-radius: $radius-m;
  }

  &.open {
    box-shadow: 0 6px 16px rgba(0, 0, 0, 0.08);
  }

// 헤더
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

 //폼그룹
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

 // 주소 입력
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

      &.disabled {
        background: #ccc;
        cursor: pointer;
        opacity: 0.7;
      }
    }
  }
}

// 에러
.error {
  color: #e53935;
  font-size: 0.85rem;
  margin-top: 4px;
  padding-left: 3px;
  line-height: 1.4;
}
// ============뷰데이픽커================
//==========외부================
.date-picker {
  width: 100%;
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
    width: 100% !important;
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

/* =========================================================
  📅 VueDatePicker 팝업 스타일
========================================================= */
:deep(.dp__outer_menu_wrap) {
  position: absolute !important;
  top: 100% !important;
  left: 50% !important;
  transform: translateX(-50%) !important;
  margin-top: 8px !important;

  z-index: 9999 !important;
  width: 350px !important;
  max-width: calc(100vw - 40px) !important;
  max-height: 90vh !important;
  overflow-y: auto !important;

  border-radius: $radius-m !important;
  background: rgba(255, 255, 255, 0.98) !important;
  border: 1px solid #d2e8e8 !important;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12) !important;
  padding: 20px !important;
}

/* ✅ 팝업 딤 제거 및 화살표 제거 */
:deep(.dp__outer_menu_wrap::before),
:deep(.dp__arrow_top),
:deep(.dp__arrow_bottom) {
  display: none !important;
}

/* =========================================================
  내부 달력 스타일
========================================================= */
:deep(.dp__menu_inner) {
  padding: 16px 0 0 0 !important; /* ✅ 좌우 패딩 제거로 버튼 꽉 차게 */
  background: #fff !important;
  border-radius: $radius-m !important;
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

/* ✅ 선택 구간 색상 */
:deep(.dp__range_start),
:deep(.dp__range_end) {
  background: $color_main !important;
  color: #fff !important;
  border-radius: $radius-s  !important;
}
:deep(.dp__range_between) {
  background: $color_main_light !important;
}

/* =========================================================
  🎨 테마 컬러 변수
========================================================= */
:deep(.dp__theme_light),
:deep(.dp__theme_dark) {
  --dp-primary-color: $color_main !important;
  --dp-primary-text-color: #fff !important;
  --dp-hover-color: $color_main_deepa !important;
  --dp-hover-text-color: #fff !important;
  --dp-success-color: $color_main !important;
  --dp-success-text-color: #fff !important;
  --dp-icon-color: $color_main !important;
  --dp-hover-icon-color: $color_main_deep !important;
  --dp-secondary-color: #f7fcfb !important;
  --dp-border-color: #d2e8e8 !important;
  --dp-menu-border-color: #d2e8e8 !important;
  --dp-range-between-dates-background-color: #eaf8f6 !important;

  /* ✅ 버튼 색상 */
  --dp-action-button-bg: $color_main !important;
  --dp-action-button-hover-bg: $color_main_deep !important;
  --dp-action-button-text-color: #fff !important;
}

/* =========================================================
  ✅ 버튼 영역 (v11 + slot 통합)
========================================================= */
:deep(.dp__action_row),
:deep(.dp__action_buttons) {
  display: flex !important;
  justify-content: center !important;
  margin: 0 !important;
  padding: 0 !important;
  width: 100% !important;
  gap: 0 !important;
}

/* ❌ 취소 버튼 숨김 */
:deep(.dp__action_cancel),
:deep(.dp__cancel) {
  display: none !important;
}

/* ✅ “선택 완료” 버튼 (기본/slot 동일 적용) */
:deep(.dp__action_select),
:deep(.dp__select),
:deep(.custom-select) {
  display: block !important;
  width: 100% !important;
  padding: 16px 0 !important;
  background: $color_main !important;
  color: #fff !important;
  text-align: center !important;
  font-weight: 700 !important;
  font-size: 1rem !important;
  border: none !important;
  border-radius: $radius-m !important;
  cursor: pointer !important;
  transition: background 0.25s ease !important;
}

:deep(.dp__action_select:hover),
:deep(.dp__select:hover),
:deep(.custom-select:hover) {
  background: $color_main !important;
}

//모바일에서만 보이는 예약전환 버튼들
.btn-group {
  display: flex;
  justify-content: space-between;
  gap: 14px;
  padding-bottom: 25px;

  .card-btn {
    flex: 1;
    border-radius: $radius-m ;
    padding: 20px 0;
    font-size: 1rem;
    font-weight: 600;
    text-align: center;
    border: none;
    cursor: pointer;
    transition: all 0.25s ease;
  }

//왼쪽 회색
  .card-btn.left {
    background: #f5f5f5;
    color: #616161;
    border: 1.5px solid #e0e0e0;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.03);

    &:hover {
      background: #eaeaea;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.08);
    }
  }

 //오른쪽 투명 메인
  .card-btn.right {
    background: rgba(83, 180, 161, 0.15);
    color: #2e7e73;
    border: 1.5px solid rgba(83, 180, 161, 0.25);
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);

    &:hover {
      background: rgba(83, 180, 161, 0.25);
      box-shadow: 0 4px 10px rgba(83, 180, 161, 0.15);
    }
  }
}


//버튼그룹 피씨에서는 안보임
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


</style>
