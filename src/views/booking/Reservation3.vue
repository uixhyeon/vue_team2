<template>
  <!-- 결제 완료 -->
  <section class="reserve-page">
    <div class="inner">
      <Stepper :current-step="3" />

      <div class="card-test">
        <div class="form_card receipt_card">
          <!-- 결제 완료 -->
          <div class="card_header">
            <h3>결제 완료</h3>
          </div>

          <div class="card_content">
            <table class="receipt_table">
              <colgroup>
                <col style="width: 65%" />
                <col style="width: 35%" />
              </colgroup>
              <tbody>
                <tr>
                  <td>결제 수단</td>
                  <td>{{ paymentLabel }}</td>
                </tr>
                <tr v-if="useCoupon">
                  <td>쿠폰 할인</td>
                  <td>- {{ formatKrw(3000) }}</td>
                </tr>
                <tr v-if="usePoints">
                  <td>포인트 사용</td>
                  <td>- {{ formatKrw(2500) }}</td>
                </tr>
                <tr class="total">
                  <td>최종 결제 금액</td>
                  <td>
                    <strong>{{ formatKrw(finalTotal) }}</strong>
                  </td>
                </tr>
              </tbody>
            </table>

            <div class="receipt_footer">
              <p>주문번호 : {{ orderId }}</p>
              <p>결제일시 : {{ formattedNow }}</p>
            </div>
          </div>

          <!-- 예약 완료 -->
          <div class="card_header"></div>

          <div class="card_content">
            <!-- 요약 보기 -->
            <div v-if="!showDetail" class="summary-view">
              <h3 class="summary-title">예약 성공!</h3>

              <div class="branch-size">
                <span>{{ form.address || "지점 미선택" }}</span>
                <span>{{ form.size || "-" }} 사이즈</span>
              </div>

              <p class="service-type">{{ selectedServices.join(" | ") }}</p>

              <div class="date-box">
                {{ formatShortDate(form.dateRange[0]) }} ~ {{ formatShortDate(form.dateRange[1]) }}
              </div>

              <div class="pickup-delivery">
                <p v-if="form.pickupDate">
                  픽업일 <span>{{ formatShortDate(form.pickupDate) }}</span>
                </p>
                <p v-if="form.deliveryDate">
                  배송요청일 <span>{{ formatShortDate(form.deliveryDate) }}</span>
                </p>
              </div>

              <p class="toggle-mini" @click="showDetail = true">상세보기 ▸</p>
            </div>

            <!-- 상세 보기 -->
            <div v-else class="detail-view">
              <h3>예약 완료</h3>
              <table class="receipt_table">
                <colgroup>
                  <col style="width: 65%" />
                  <col style="width: 35%" />
                </colgroup>
                <tbody>
                  <tr v-if="form.name">
                    <td>성함</td>
                    <td>{{ form.name }}</td>
                  </tr>
                  <tr v-if="form.phone">
                    <td>휴대폰</td>
                    <td>{{ form.phone }}</td>
                  </tr>
                  <tr v-if="form.size">
                    <td>사물함 사이즈</td>
                    <td>{{ form.size }}</td>
                  </tr>
                  <tr v-if="form.address">
                    <td>대여 장소</td>
                    <td>{{ form.address }}</td>
                  </tr>
                  <tr v-if="form.dateRange && form.dateRange[0] && form.dateRange[1]">
                    <td>예약 기간</td>
                    <td>{{ formatDate(form.dateRange[0]) }} ~ {{ formatDate(form.dateRange[1]) }}</td>
                  </tr>
                  <tr v-if="form.pickupAddress">
                    <td>픽업 주소</td>
                    <td>{{ form.pickupAddress }}</td>
                  </tr>
                  <tr v-if="form.pickupAddressDetail">
                    <td>상세 주소</td>
                    <td>{{ form.pickupAddressDetail }}</td>
                  </tr>
                  <tr v-if="form.pickupDate">
                    <td>픽업일</td>
                    <td>{{ formatDate(form.pickupDate) }}</td>
                  </tr>
                  <tr v-if="form.homeAddress">
                    <td>배송 주소</td>
                    <td>{{ form.homeAddress }}</td>
                  </tr>
                  <tr v-if="form.homeAddressDetail">
                    <td>상세 주소</td>
                    <td>{{ form.homeAddressDetail }}</td>
                  </tr>
                  <tr v-if="form.deliveryDate">
                    <td>배송일</td>
                    <td>{{ formatDate(form.deliveryDate) }}</td>
                  </tr>
                </tbody>
              </table>

              <p class="toggle-mini" @click="showDetail = false">닫기 ▴</p>
            </div>
          </div>

          <!-- 큐알표시 -->
          <div class="qr-section always">
            <img :src="qrImage" alt="예약 QR코드" class="qr-thumb" @click="showQRModal = true" />
            <p class="qr-desc">예약 QR코드</p>
            <p class="qr-desc">예약번호: {{ orderId }}</p>
          </div>
        </div>

        <button class="submit_btn" @click="goToHome">홈으로 이동</button>
      </div>
    </div>

    <!-- 큐알 확대 모달 -->
    <transition name="fade">
      <div v-if="showQRModal" class="qr-modal" @click.self="showQRModal = false">
        <div class="qr-modal-content">
          <img :src="qrImage" alt="예약 QR코드" class="qr-large" />
          <div class="qr-btn-row">
            <button @click="downloadQR">다운로드</button>
            <button @click="showQRModal = false">닫기</button>
          </div>
        </div>
      </div>
    </transition>
  </section>
</template>

<script setup>
import { useRoute, useRouter } from "vue-router";
import { ref, computed } from "vue";
import Stepper from "@/components/reserv/Stepper.vue";

const route = useRoute();
const router = useRouter();

// 예약받기
const orderId = ref(route.query.orderId || "");

// 전달된 데이터 받기
const form = ref(
  route.query.form
    ? JSON.parse(route.query.form)
    : {
        name: "",
        phone: "",
        size: "",
        address: "",
        dateRange: [],
        pickupAddress: "",
        pickupAddressDetail: "",
        pickupDate: "",
        homeAddress: "",
        homeAddressDetail: "",
        deliveryDate: "",
      }
);

const useCoupon = ref(route.query.useCoupon === "true");
const usePoints = ref(route.query.usePoints === "true");
const selectedPayment = ref(route.query.payment || "card");
const total = Number(route.query.total) || 0;

// 결제 수단명
const paymentLabel = computed(() => {
  switch (selectedPayment.value) {
    case "card":
      return "💳 신용카드";
    case "kakao":
      return "💬 카카오페이";
    case "naver":
      return "N Pay";
    case "bank":
      return "🏦 무통장입금";
    default:
      return "-";
  }
});

// 할인 계산
const discount = computed(() => {
  let d = 0;
  if (useCoupon.value) d += 3000;
  if (usePoints.value) d += 2500;
  return d;
});

const finalTotal = computed(() => total);

// 통화 포맷
const formatKrw = (v) => new Intl.NumberFormat("ko-KR", { style: "currency", currency: "KRW" }).format(v);

// 현재 시각
const formattedNow = new Date().toLocaleString("ko-KR", {
  year: "numeric",
  month: "2-digit",
  day: "2-digit",
  hour: "2-digit",
  minute: "2-digit",
});

// 홈으로 이동
const goToHome = () => {
  router.push("/");
};

// 모달띄우기
const showQRModal = ref(false);
const qrImage = "/images/reservation/qrcode.png"; // 실제 QR 이미지 경로

const downloadQR = () => {
  const link = document.createElement("a");
  link.href = qrImage;
  link.download = "예약_QR.png";
  link.click();
};

// 날짜 포맷 함수 제일 밑에 둑기

const formatDate = (date) => {
  if (!date) return "";
  const d = new Date(date);
  return d.toLocaleDateString("ko-KR", {
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
  });
};

// 요약 / 상세보기 상태
const showDetail = ref(false);

// 선택된 서비스 계산
const selectedServices = computed(() => {
  const list = [];
  if (form.value.size) list.push("사물함 대여");
  if (form.value.pickupAddress) list.push("짐 가져오기");
  if (form.value.homeAddress) list.push("짐 배송하기");
  return list;
});

// 짧은 날짜 포맷
const formatShortDate = (date) => {
  if (!date) return "";
  const d = new Date(date);
  const y = String(d.getFullYear()).slice(2);
  const m = d.getMonth() + 1;
  const day = d.getDate();
  return `${y}.${m}.${day}`;
};
</script>
<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

// ===========폰트크기설정================
/* // 메인제목 : 20~22px */
.main-title,
.card_header h3 {
  font-size: clamp(1.25rem, 1.8vw, 1.375rem);
  font-weight: 700;
  color: #222;
  line-height: 1.4;
  margin-bottom: 1rem;
}

/* // 소제목 : 18~20px */
.section-title,
.summary-title {
  font-size: clamp(1.125rem, 1.5vw, 1.25rem);
  font-weight: 600;
  color: #333;
  line-height: 1.5;
  margin-bottom: 0.8rem;
}

/* // 본문텍스트 : 16~18px */
.body-text,
.receipt_table td,
.pickup-delivery,
.date-box {
  font-size: clamp(1rem, 1.1vw, 1.125rem);
  color: #444;
  line-height: 1.6;
  word-break: keep-all;
}

/* // 부가텍스트 : 14~15px */
.sub-text,
.qr-desc,
.toggle-mini,
.receipt_footer {
  font-size: clamp(0.875rem, 0.9vw, 0.95rem);
  color: #777;
  line-height: 1.5;
}

//배경부터
.reserve-page {
  background: #f5f7f7;
  padding: 40px 0 80px;
}

.inner {
  width: 100%;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
}

//====공통================
.form_card {
  background: #fff;
  border-radius: $radius-m;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.06);
  border: 1px solid #fff;
  position: relative;
  width: 100%;
  max-width: 650px;
  padding: clamp(20px, 3vw, 28px) clamp(16px, 4vw, 24px);
  box-sizing: border-box;
  transition: padding 0.2s ease, font-size 0.2s ease;
  padding: 20px;

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

  .card_header h3 {
    letter-spacing: -0.2px;
  }

  .card_content {
    color: #444;
    line-height: 1.6;
    word-break: keep-all;
  }
}

//====결제완료 테이블================
.receipt_card {
  text-align: center;

  .receipt_table {
    width: 100%;
    border-collapse: collapse;
    margin-bottom: 1.5rem;

    tr {
      border-bottom: 1px solid #eee;

      td {
        padding: clamp(8px, 1vw, 10px) 0;
        text-align: left;

        &:first-child {
          width: 45%;
          color: #777;
        }

        &:last-child {
          text-align: right;
        }
      }

      &.total td:last-child {
        font-weight: 600;
        color: $color_main;
      }
    }
  }

  .receipt_footer {
    text-align: left;
    border-top: 1px dashed #ddd;
    padding-top: 10px;
    margin-top: 10px;

    p {
      margin: 3px 0;
    }
  }
}

//====공통 버튼================
.submit_btn {
  width: 80%;
  margin-top: 20px;
  max-width: 300px;
  padding: 14px 0;
  font-size: $button;
  font-weight: 600;
  color: #fff;
  background: $color_main;
  border: none;
  border-radius: $radius-s;
  cursor: pointer;
  transition: background 0.2s ease;

  &:hover {
    background: $color_main_deep;
  }
}

//====카드 컨테이너================
.card-test {
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: #f5f7f7;
}

//====QR 코드================
.qr-section {
  text-align: center;
  margin-top: clamp(24px, 3vw, 32px);
  margin-bottom: 10px;

  &.always {
    margin-top: 30px;
  }

  .qr-thumb {
    width: clamp(90px, 12vw, 120px);
    height: clamp(90px, 12vw, 120px);
    border-radius: $radius-s;
    border: 1px solid #ddd;
    background: #fff;
    cursor: pointer;
    transition: transform 0.2s ease;

    &:hover {
      transform: scale(1.05);
    }
  }
}

//====QR 모달================
.qr-modal {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}

.qr-modal-content {
  background: #fff;
  border-radius: $radius-m;
  padding: 24px;
  text-align: center;
  max-width: 360px;
  width: 90%;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);

  .qr-large {
    width: clamp(200px, 30vw, 250px);
    height: clamp(200px, 30vw, 250px);
    border-radius: $radius-m;
    border: 1px solid #e7e7e7;
  }

  .qr-btn-row {
    margin-top: 16px;
    display: flex;
    justify-content: center;
    gap: 10px;

    button {
      padding: 8px 14px;
      border: none;
      border-radius: $radius-s;
      cursor: pointer;
      font-size: 0.9rem;
      font-weight: 500;
      transition: background 0.2s ease;

      &:first-child {
        background: $color_main;
        color: #fff;

        &:hover {
          background: $color_main_deep;
        }
      }

      &:last-child {
        background: #eee;

        &:hover {
          background: #ddd;
        }
      }
    }
  }
}

//====토글 버튼================
.toggle-mini {
  margin-top: 10px;
  cursor: pointer;
  text-align: right;
  padding-right: 1rem;
  transition: color 0.2s ease;

  &:hover {
    color: $color_main;
  }
}

//====@사라진것 다시 살리기@================
.summary-view {
  width: 100%;

  &::before {
    content: "😊";
    display: block;
    font-size: 2.5rem;
    margin-bottom: 0.8rem;
  }

  .branch-size {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-bottom: 0.8rem;

    span {
      border: 1px solid #ccc;
      border-radius: $radius-l;
      padding: clamp(4px, 1vw, 6px) clamp(12px, 3vw, 20px);
      background: #fff;
    }
  }
}

//====모달 애니메이션================
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

//====미디어 쿼리================
@media (max-width: 600px) {
  .form_card {
    width: 90%;

    .receipt_table td {
      padding: 6px 0;
    }

    .summary-title {
      font-size: 1.05rem !important;
    }

    .qr-thumb {
      width: 90px !important;
      height: 90px !important;
    }

    .branch-size span {
      padding: 4px 12px !important;
    }
  }

  .submit_btn {
    width: 70%;
    max-width: none;
  }
}

//====요약 보기================
.summary-view {
  width: 100%;

  &::before {
    content: "😊";
    display: block;
    font-size: 2.5rem;
    margin-bottom: 0.8rem;
  }

  .summary-title {
    font-size: clamp(1.1rem, 1.4vw, 1.3rem);
    font-weight: 700;
    color: $color_main;
    margin-bottom: 1.2rem;
  }

  .branch-size {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-bottom: 0.8rem;

    span {
      border: 1px solid #ccc;
      border-radius: $radius-l;
      padding: clamp(4px, 1vw, 6px) clamp(12px, 3vw, 20px);
      background: #fff;
    }
  }

  .service-type {
    color: #555;
    font-size: clamp(0.85rem, 1vw, 0.9rem);
    margin-bottom: 1rem;
  }

  /* 여기 추가 */
  .date-box {
    background: $color_main_background;
    border: 1px solid rgba(0, 0, 0, 0.05);
    border-radius: $radius-s;
    color: #222;
    font-weight: 600;
    font-size: clamp(0.9rem, 1.1vw, 1rem);
    display: inline-block;
    padding: clamp(10px, 1vw, 12px) clamp(14px, 2vw, 16px);
    margin-bottom: 1.2rem;
    box-shadow: inset 0 0 3px rgba(0, 0, 0, 0.03);
  }

  .pickup-delivery {
    font-size: clamp(0.8rem, 1vw, 0.9rem);
    color: #444;
    margin-bottom: 0.5rem;
    line-height: 1.6;

    p {
      display: flex;
      justify-content: center;
      gap: 10px;

      span {
        font-weight: 600;
        color: #111;
      }
    }
  }
}
</style>
