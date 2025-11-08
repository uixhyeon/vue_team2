<template>
  <div class="wrap">
    <Stepper :current-step="currentStep" />

    <!-- ✅ 통일된 배경/레이아웃 -->
    <div class="background inner">
      <div class="container">
        <!-- 왼쪽 입력 카드들 -->
        <div class="left">
          <!-- ① 사물함 예약 -->
          <Reserv1Locker
            v-model:form="form"
            :isOpen="openSection === 'locker'"
            :errors="errors"
            :touched="touched"
            @toggle="toggleSection('locker')"
            @openBranch="handleOpenBranch"
            @touch="handleTouch"
            @move="handleMove"
              v-model:selectedBranch="selectedBranch"
          />

          <!-- ② 짐 가져오기 -->
          <Reserv2Arrival
            v-model:form="form"
            :isOpen="openSection === 'arrival'"
            :errors="errors"
            :touched="touched"
            @toggle="toggleSection('arrival')"
            @openPickup="openPickupAddr = true"
            @touch="handleTouch"
            @move="handleMove"
          />

          <!-- ③ 집으로 보내기 -->
          <Reserv3Luggage
            v-model:form="form"
            :isOpen="openSection === 'luggage'"
            :errors="errors"
            :touched="touched"
            @toggle="toggleSection('luggage')"
            @openHome="openHomeAddr = true"
            @move="handleMove"
          />
        </div>

        <!-- 오른쪽 요약 카드 -->
        <div class="right">
          <Reserv4Summary
            :form="form"
            :totalPrice="totalPrice"
            :hasInput="hasInput"
            :lockerComplete="lockerComplete"
            :arrivalComplete="arrivalComplete"
            :luggageComplete="luggageComplete"
            :errors="errors"
            @submit="handleSubmit"
          />

          <!-- ✅ 입력 완료 버튼 -->
          <button class="submit_btn" @click="handleSubmit">입력 완료</button>
        </div>
        <button class="mobile-submit" @click="handleMobileComplete">입력 완료</button>
      </div>
    </div>
    <!-- ===== 모달들 ===== -->
    <BranchSelectModal
      :open="showBranchModal"
      :locations="locations"
       :selectedBranch="selectedBranch"   
      @close="showBranchModal = false"
      @selectBranch="handleBranchSelect"
    />

    <AddressPicker
      v-model="form.pickupAddress"
      :open="openPickupAddr"
      @close="openPickupAddr = false"
      @selected="(addr) => (form.pickupAddress = addr)"
    />

    <AddressPicker
      v-model="form.homeAddress"
      :open="openHomeAddr"
      @close="openHomeAddr = false"
      @selected="(addr) => (form.homeAddress = addr)"
    />

    <!-- 💚 MatAju 전역 알림창 -->
    <!-- 💚 AlertModal (body로 이동됨, 렌더 순서 영향 없음) -->
    <AlertModal :show="showAlert" :message="alertMessage" @close="showAlert = false" />
    <!-- 컨팜모달 -->
    <ConfirmReserv
      v-if="showConfirm && isTabletOrBelow"
      :form="form"
      :total-price="totalPrice"
      @close="showConfirm = false"
      @submit="handleConfirmSubmit"
    />
    <!--약관동의 -->
    <ReasrAgree :show="showTerms" @close="showTerms = false" @agree="handleAgreeTerms" />
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from "vue";
import { useRouter } from "vue-router";

import Stepper from "@/components/reserv/Stepper.vue";
import ReasrAgree from "@/components/reserv/ReserAgree.vue";

import Reserv1Locker from "@/views/booking/Reserv1Locker.vue";

import Reserv2Arrival from "@/views/booking/Reserv2Arrival.vue";
import Reserv3Luggage from "@/views/booking/Reserv3Luggage.vue";
import Reserv4Summary from "@/views/booking/Reserv4Summary.vue";

import BranchSelectModal from "@/components/reserv/BranchSelectModal.vue";

import AddressPicker from "@/components/reserv/AddressPicker.vue";
import ConfirmReserv from "@/components/reserv/ConfirmReserv.vue";

// 💚 추가된 전역 알림창 상태
const showAlert = ref(false);
const alertMessage = ref("");

// 약관동의
const showTerms = ref(false);

// 약관 동의 완료 처리
function handleAgreeTerms() {
  showTerms.value = false;

  alertMessage.value = "이용약관에 동의하였습니다.";
  showAlert.value = true;

  setTimeout(() => {
    router.push({
      path: "/reservation2",
      query: {
        form: JSON.stringify(form.value),
        totalPrice: totalPrice.value,
      },
    });
  }, 1200);
}

// 스텝퍼 - 항상 1단계 유지
const currentStep = computed(() => 1);

// 📍 지역별 지점 리스트
const locations = [
  {
    region: "부산 광안리",
    branches: [
      {
        id: 1,
        name: "광안리 해변점",
        address: "부산광역시 수영구 광안해변로 203",
        lat: 35.1531,
        lng: 129.1187,
        lockers: "3개 남음",
        status: "운영중",
      },
      {
        id: 2,
        name: "광안시장점",
        address: "부산광역시 수영구 남천동로 12-1", // 📍 실제 존재 주소
        lat: 35.1475,
        lng: 129.1092,
        lockers: "5개 남음",
        status: "운영중",
      },
      {
        id: 3,
        name: "광안역점",
        address: "부산광역시 수영구 광안로 45",
        lat: 35.1556,
        lng: 129.1139,
        lockers: "4개 남음",
        status: "점검중",
      },
    ],
  },
  {
    region: "강릉시",
    branches: [
      {
        id: 4,
        name: "강릉역점",
        address: "강원특별자치도 강릉시 용지로 123",
        lat: 37.7642,
        lng: 128.8997,
        lockers: "6개 남음",
        status: "운영중",
      },
      {
        id: 5,
        name: "경포해변점",
        address: "강원특별자치도 강릉시 창해로 240-3",
        lat: 37.7956,
        lng: 128.9152,
        lockers: "7개 남음",
        status: "운영중",
      },
    ],
  },
  {
    region: "속초",
    branches: [
      {
        id: 6,
        name: "속초중앙시장점",
        address: "강원특별자치도 속초시 중앙로 147",
        lat: 38.2073,
        lng: 128.5912,
        lockers: "2개 남음",
        status: "운영중",
      },
      {
        id: 7,
        name: "속초해수욕장점",
        address: "강원특별자치도 속초시 해오름로 190",
        lat: 38.1792,
        lng: 128.6094,
        lockers: "7개 남음",
        status: "점검중",
      },
    ],
  },
  {
    region: "전주",
    branches: [
      {
        id: 8,
        name: "전주한옥마을점",
        address: "전라북도 전주시 완산구 기린대로 99",
        lat: 35.8151,
        lng: 127.1527,
        lockers: "2개 남음",
        status: "운영중",
      },
    ],
  },
  {
    region: "제주도",
    branches: [
      {
        id: 9,
        name: "제주시청점",
        address: "제주특별자치도 제주시 관덕로 9",
        lat: 33.5113,
        lng: 126.5207,
        lockers: "2개 남음",
        status: "운영중",
      },
      {
        id: 10,
        name: "서귀포점",
        address: "제주특별자치도 서귀포시 중문관광로 72",
        lat: 33.2540,
        lng: 126.4146,
        lockers: "6개 남음",
        status: "운영중",
      },
    ],
  },
  {
    region: "오사카",
    branches: [
      {
        id: 11,
        name: "난바역점",
        address: "Namba Station, Osaka, Japan",
        lat: 34.6667,
        lng: 135.5010,
        lockers: "3개 남음",
        status: "운영중",
      },
      {
        id: 12,
        name: "우메다점",
        address: "2-14-7 Sonezaki, Kita Ward, Osaka, Japan",
        lat: 34.7033,
        lng: 135.4983,
        lockers: "5개 남음",
        status: "운영중",
      },
    ],
  },
];


// ✅ 추가: 선택된 지점 상태
const selectedBranch = ref(null);

// ✅ 공통 폼 상태
const form = ref({
  name: "",
  phone: "",
  size: "",
  address: "",
  dateRange: null,
  pickupAddress: "",
  pickupAddressDetail: "",
  pickupDate: "",
  homeAddress: "",
  homeAddressDetail: "",
  deliveryDate: "",
});

// ✅ 가격표
const prices = {
  S: { locker: 5000, delivery: 4000 },
  M: { locker: 8000, delivery: 6000 },
  L: { locker: 15000, delivery: 14000 },
  XL: { locker: 20000, delivery: 22000 },
  XXL: { locker: 28000, delivery: 32000 },
};

// ✅ 완료 상태
const lockerComplete = computed(() => {
  const f = form.value;
  return f.name && f.phone && f.size && f.address && f.dateRange && f.dateRange[0] && f.dateRange[1];
});

// ✅ 열기/닫기
const openSection = ref("locker");
const toggleSection = (name) => {
  const f = form.value;

  // ⚠ 기존 alert
  // alert("먼저 사물함 예약을 완료해주세요.");
  // 💚 새 알림창
  if (!lockerComplete.value && name !== "locker") {
    alertMessage.value = "먼저 사물함 예약을 완료해주세요.";
    showAlert.value = true;
    return;
  }

  const prev = openSection.value;
  if (prev === "arrival") {
    const filled = f.pickupAddress?.trim() && f.pickupAddressDetail?.trim() && f.pickupDate;
    if (!filled) {
      f.pickupAddress = "";
      f.pickupAddressDetail = "";
      f.pickupDate = "";
    }
  }
  if (prev === "luggage") {
    const filled = f.homeAddress?.trim() && f.homeAddressDetail?.trim() && f.deliveryDate;
    if (!filled) {
      f.homeAddress = "";
      f.homeAddressDetail = "";
      f.deliveryDate = "";
    }
  }
  openSection.value = openSection.value === name ? null : name;
  form.value = { ...f };
};

function handleOpenBranch() {
  if (!form.value.size) {
    // ⚠ 기존: alert("사물함 사이즈를 먼저 선택해주세요.");
    // ✅ 수정: 전역 AlertModal 호출
    alertMessage.value = "사물함 사이즈를 먼저 선택해주세요.";
    showAlert.value = true;
    return;
  }

  showBranchModal.value = true; // ✅ 정상적으로 모달 열림
}

//모바일 컨펌레져 모달띠우기
function handleConfirmSubmit() {
  showConfirm.value = false;
  handleSubmit();
}

// ✅ 완료 체크
const arrivalComplete = computed(() => {
  const f = form.value;
  return f.pickupAddress && f.pickupAddressDetail && f.pickupDate;
});
const luggageComplete = computed(() => {
  const f = form.value;
  return f.homeAddress && f.homeAddressDetail && f.deliveryDate;
});

// ✅ 에러 상태
const errors = ref({});
const touched = ref({
  name: false,
  phone: false,
  size: false,
  address: false,
  dateRange: false,
  pickupAddress: false,
  pickupAddressDetail: false,
  pickupDate: false,
  homeAddress: false,
  homeAddressDetail: false,
  deliveryDate: false,
});

function handleTouch(field) {
  touched.value[field] = true;
}

// ✅ 유효성 검사
const validateForm = () => {
  const f = form.value;
  const err = {};
  if (!f.name?.trim()) err.name = "이름을 입력해주세요";
if (!f.phone || !/^010\d{8}$/.test(f.phone)) 
  err.phone = "010으로 시작하는 11자리 숫자를 입력해주세요 (- 제외)";
  if (!f.size) err.size = "사물함 사이즈를 선택해주세요";
  if (!f.address) err.address = "대여 장소를 선택해주세요";
  if (!f.dateRange || f.dateRange.length < 2 || f.dateRange[0] === f.dateRange[1])
    err.dateRange = "시작일과 종료일이 같을 수 없습니다.";

  if (f.pickupAddress || f.pickupAddressDetail || f.pickupDate) {
    if (!f.pickupAddress) err.pickupAddress = "픽업 주소를 선택해주세요";
    if (!f.pickupAddressDetail) err.pickupAddressDetail = "픽업 상세주소를 입력해주세요";
    if (!f.pickupDate) err.pickupDate = "픽업일을 선택하고 확인을 눌러주세요";
  }
  if (f.homeAddress || f.homeAddressDetail || f.deliveryDate) {
    if (!f.homeAddress) err.homeAddress = "배송 주소를 선택해주세요";
    if (!f.homeAddressDetail) err.homeAddressDetail = "배송 상세주소를 입력해주세요";
    if (!f.deliveryDate) err.deliveryDate = "배송일을 선택하고 확인을 눌러주세요";
  }

  errors.value = err;
  return Object.keys(err).length === 0;
};
// ✅ 실시간 감시
watch(
  form,
  (f) => {
    const err = {};

    if (!f.name?.trim()) err.name = "이름을 입력해주세요";
   if (!f.phone || !/^010\d{8}$/.test(f.phone)) 
  err.phone = "숫자 11자리를 입력해주세요 (- 제외)";

    if (!f.size) err.size = "사물함 사이즈를 선택해주세요";
    if (!f.address) err.address = "대여 장소를 선택해주세요";

    // ✅ dateRange 보강 (원래 로직 + 추가 조건)
    if (!f.dateRange || f.dateRange.length < 2) {
      err.dateRange = "예약 기간을 선택해주세요";
    } else if (f.dateRange[0] && f.dateRange[1] && f.dateRange[0] === f.dateRange[1]) {
      err.dateRange = "시작일과 종료일이 같을 수 없습니다.";
    }

    if (f.pickupAddress || f.pickupAddressDetail || f.pickupDate) {
      if (!f.pickupAddress) err.pickupAddress = "픽업 주소를 선택해주세요";
      if (!f.pickupAddressDetail) err.pickupAddressDetail = "픽업 상세주소를 입력해주세요";
      if (!f.pickupDate) err.pickupDate = "픽업일을 선택하고 확인 버튼을 눌러주세요";
    }

    if (f.homeAddress || f.homeAddressDetail || f.deliveryDate) {
      if (!f.homeAddress) err.homeAddress = "배송 주소를 선택해주세요";
      if (!f.homeAddressDetail) err.homeAddressDetail = "배송 상세주소를 입력해주세요";
      if (!f.deliveryDate) err.deliveryDate = "배송일을 선택하고 확인 버튼을 눌러주세요";
    }

    errors.value = err;
  },
  { deep: true, flush: "post" } // ✅ flush 보강
);

// ✅ 입력 감지
const hasInput = computed(() => {
  const f = form.value;
  return f.name || f.phone || f.size || f.address || f.dateRange || f.pickupAddress || f.homeAddress;
});

// ✅ 모달 관리
const showBranchModal = ref(false);
const openPickupAddr = ref(false);
const openHomeAddr = ref(false);

function handleBranchSelect(location) {
  form.value.address = location.name;
    selectedBranch.value = location; // ✅ 선택 지점 전체 저장
  console.log(" 선택된 지점:", location); // ✅ 연결 확인용 테스트용
  showBranchModal.value = false;
}

// ✅ 총 요금 계산
const rentalDays = computed(() => {
  const r = form.value.dateRange;
  if (!r || r.length < 2) return 0;
  const s = new Date(r[0]),
    e = new Date(r[1]);
  return (e - s) / (1000 * 60 * 60 * 24) + 1;
});

const totalPrice = computed(() => {
  const f = form.value;
  const s = f.size || "";
  let t = 0;
  if (lockerComplete.value) t += (prices[s]?.locker ?? 0) * rentalDays.value;
  if (arrivalComplete.value) t += prices[s]?.delivery ?? 0;
  if (luggageComplete.value) t += prices[s]?.delivery ?? 0;
  return t;
});

// ✅ 제출
const router = useRouter();
const handleSubmit = () => {
  // ⚠ 기존 alert
  // alert("입력값을 다시 확인해주세요.");
  // 💚 새 알림창
  if (!validateForm()) {
    alertMessage.value = "입력값을 다시 확인해주세요.";
    showAlert.value = true;

    if (errors.value.name || errors.value.phone || errors.value.size || errors.value.address || errors.value.dateRange)
      openSection.value = "locker";
    else if (errors.value.pickupAddress || errors.value.pickupAddressDetail || errors.value.pickupDate)
      openSection.value = "arrival";
    else if (errors.value.homeAddress || errors.value.homeAddressDetail || errors.value.deliveryDate)
      openSection.value = "luggage";
    return;
  }
  showTerms.value = true; // ✅ 약관 모달 열기
};
// 모바일 카드분리
// 카드 순서 배열
// 카드 순서 배열 (유지)
const sectionOrder = ["locker", "arrival", "luggage"];

// ✅ 공통 이동 핸들러 (단일 인자 version)
function handleMove(target) {
  if (!target) return;

  // ① 사물함이 완성되지 않은 상태에서 다른 카드로 이동하려고 하면 막기
  if (target !== "locker" && !lockerComplete.value) {
    alertMessage.value = "먼저 사물함 예약을 완료해주세요.";
    showAlert.value = true;
    return;
  }

  const f = form.value;
  const prev = openSection.value;

  // ② 이전 카드가 arrival인데, 값이 덜 채워진 상태로 넘어가면 초기화
  if (prev === "arrival" && target !== "arrival") {
    const filled = f.pickupAddress?.trim() && f.pickupAddressDetail?.trim() && f.pickupDate;
    if (!filled) {
      f.pickupAddress = "";
      f.pickupAddressDetail = "";
      f.pickupDate = "";
    }
  }

  // ③ 이전 카드가 luggage인데, 값이 덜 채워진 상태로 넘어가면 초기화
  if (prev === "luggage" && target !== "luggage") {
    const filled = f.homeAddress?.trim() && f.homeAddressDetail?.trim() && f.deliveryDate;
    if (!filled) {
      f.homeAddress = "";
      f.homeAddressDetail = "";
      f.deliveryDate = "";
    }
  }

  // ④ target이 sectionOrder 안에 있으면 해당 카드로 이동
  if (sectionOrder.includes(target)) {
    openSection.value = target;
  }

  // ⑤ reactive 갱신
  form.value = { ...f };
}

// ============모바일 추가======
// Reservation.vue
const showConfirm = ref(false); // ✅ 확인창 표시 상태

// 1024이하에서
// 지금 있는 것들 위/아래 아무데나 적당히
const windowWidth = ref(0);
const isTabletOrBelow = computed(() => windowWidth.value <= 1024);

// 창 크기 바뀔 때마다 이걸로 갱신
const updateWidth = () => {
  // SSR 대비로 window 있는지 체크
  if (typeof window !== "undefined") {
    windowWidth.value = window.innerWidth;
  }
};

onMounted(() => {
  updateWidth(); // 처음 들어왔을 때 한 번
  window.addEventListener("resize", updateWidth);
});

onUnmounted(() => {
  window.removeEventListener("resize", updateWidth);
});

function handleMobileComplete() {
  // 1024보다 크면 → PC 플로우
  if (!isTabletOrBelow.value) {
    handleSubmit();
    return;
  }

  // 1024 이하면서 → 컨펌 모달 열기
  if (!lockerComplete.value) {
    alertMessage.value = "사물함 예약 정보를 먼저 입력해주세요.";
    showAlert.value = true;
    return;
  }

  showConfirm.value = true; // ✅ 이때만 모달
}
</script>
<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

.wrap {
  background: #f5f7f7;
  padding: 40px 0 100px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.background.inner {
  width: 100%;
  max-width: 1120px;
  margin: 0 auto;
  padding: 0 40px;
  box-sizing: border-box;
}

.container {
  display: grid;
  grid-template-columns: 3fr 2fr;
  gap: 2.5rem;
  align-items: flex-start;
  width: 100%;
}

.left {
  display: flex;
  flex-direction: column;
  gap: 1.3rem;
}

.right {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.3rem;
  align-self: flex-start;
}

.form_card,
.summary_card {
  width: 100%;
  background: #fff;
  border-radius: $radius-m;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  border: 1px solid #f0f0f0;
  transition: all 0.25s ease;
  position: relative;
  z-index: 1;

  /* DatePicker가 카드 위에 표시되도록 */
  &:has(.date-picker) {
    overflow: visible;
  }
}

.form_card.open {
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.08);
  transform: translateY(-1px);
}

.submit_btn {
  width: 70%;
  padding: 14px 0;
  font-weight: 600;
  font-size: $button;
  color: #fff;
  background: $color_main;
  border: none;
  border-radius: $radius-s;
  cursor: pointer;
  transition: 0.3s ease;
  margin: 20px auto 0;
  display: block;

  &:hover {
    background: $color_main_deep;
  }
}

@media (max-width: 1024px) {
  .container {
    grid-template-columns: 1fr;
    gap: 2rem;
    max-width: 700px;
    margin: 0 auto; /* 중앙정렬 */
  }
  .submit_btn {
    width: 90%;
  }
}

@media (max-width: 480px) {
  .background.inner {
    padding: 0 20px;
  }
  .container {
    gap: 1.3rem;
  }
  .submit_btn {
    width: 100%;
    font-size: 0.95rem;
    padding: 12px 0;
  }
}

/* =========================================================
  ✅ Tablet 이하에서만 단계형 카드 전환 활성화
========================================================= */
@media (max-width: 1024px) {
  .left {
    display: flex;
    flex-direction: column;
    position: relative; /* 추가 */

    .form_card {
      display: none; // 기본 숨김
      opacity: 0;
      transition: all 0.3s ease;
      position: relative; /* 추가 */
      z-index: 1; /* 추가 */
    }

    .form_card.open {
      display: block;
      opacity: 1;
      animation: fadeSlide 0.3s ease forwards;
      z-index: 2; /* 열린 카드는 더 높은 z-index */
    }
  }

  @keyframes fadeSlide {
    from {
      opacity: 0;
      transform: translateY(12px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
}
//=================모바일=========
/* Reservation.vue style 영역 맨 아래 */
.mobile-submit {
  display: none;
}

@media (max-width: 1024px) {
  .right {
    display: none;
  } /* ✅ 요약카드 숨기기 */

  .mobile-submit {
    display: block;
    width: 100%;
    margin-top: 1rem;
    background: $color_main;
    color: #fff;
    border: none;
    border-radius: $radius-s;
    padding: 14px 0;
    font-weight: 600;
    text-align: center;
  }
}

/* ✅ 확인 모달 */
.confirm-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.35);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}
.confirm-box {
  background: #fff;
  border-radius: $radius-s;
  padding: 20px;
  width: min(90vw, 400px);
}
</style>
