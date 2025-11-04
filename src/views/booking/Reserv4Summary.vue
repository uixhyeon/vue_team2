<template>
  <div class="summary_card">
    <h2 class="card_title">선택 상품 요약</h2>

    <!-- 입력 전 -->
    <div v-if="!hasInput" class="summary-guide">
      원하시는 상품을 선택해주세요 🧳
    </div>

    <!-- 입력 후 -->
    <table v-else>
      <tbody>
        <!-- 사물함 예약 -->
        <template
          v-if="lockerComplete || form.name || form.phone || form.size || form.address || form.dateRange"
        >
          <tr class="section-title">
            <td colspan="2" class="s-title">사물함 예약</td>
          </tr>
          <tr v-if="form.name"><td>성함</td><td>{{ form.name }}</td></tr>
          <tr v-if="form.phone"><td>휴대폰</td><td>{{ form.phone }}</td></tr>
          <tr v-if="form.size"><td>사물함 사이즈</td><td>{{ form.size }}</td></tr>
          <tr v-if="form.address"><td>대여 장소</td><td>{{ form.address }}</td></tr>
       
        </template>
<!-- 짐 가져오기 -->
<template v-if="form.pickupAddress || form.pickupAddressDetail || form.pickupDate">
  <tr class="section-title">
    <td colspan="2" class="s-title">짐 가져오기</td>
  </tr>
  <tr v-if="form.pickupAddress">
    <td>픽업 주소</td>
    <td>{{ form.pickupAddress }}</td>
  </tr>
  <tr v-if="form.pickupAddressDetail">
    <td>상세주소</td>
    <td>{{ form.pickupAddressDetail }}</td>
  </tr>
  <tr v-if="form.pickupDate">
    <td>픽업일</td>
    <td>{{ formatDate(form.pickupDate) }}</td>
  </tr>
</template>

<!-- 집으로 보내기 -->
<template v-if="form.homeAddress || form.homeAddressDetail || form.deliveryDate">
  <tr class="section-title">
    <td colspan="2" class="s-title">집으로 보내기</td>
  </tr>
  <tr v-if="form.homeAddress">
    <td>배송 주소</td>
    <td>{{ form.homeAddress }}</td>
  </tr>
  <tr v-if="form.homeAddressDetail">
    <td>상세주소</td>
    <td>{{ form.homeAddressDetail }}</td>
  </tr>
  <tr v-if="form.deliveryDate">
    <td>배송일</td>
    <td>{{ formatDate(form.deliveryDate) }}</td>
  </tr>
</template>

        <!-- 총 금액 -->
        <tr v-if="totalPrice > 0" class="total">
          <td>총 결제금액</td>
          <td><strong>{{ formatKrw(totalPrice) }}</strong></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
defineProps({
  form: Object,
  totalPrice: Number,
  hasInput: Boolean,
  lockerComplete: Boolean,
  arrivalComplete: Boolean,
  luggageComplete: Boolean,
});

// 뷰데이 추가
const formatDate = (date) => {
  if (!date) return "";

  try {
    // VueDatePicker는 종종 ISO 문자열을 반환하므로 여기서 Date로 변환
    const parsed =
      typeof date === "string"
        ? new Date(date)
        : date instanceof Date
        ? date
        : new Date(String(date));

    // 변환 실패 시 원본 반환
    if (isNaN(parsed)) return String(date);

    // 한국시간(KST) 기준 날짜 문자열로 변환
    const local = new Date(parsed.getTime() + 9 * 60 * 60 * 1000);

    return local.toLocaleDateString("ko-KR", {
      year: "numeric",
      month: "2-digit",
      day: "2-digit",
    });
  } catch (err) {
    console.warn("날짜 포맷 변환 실패:", date, err);
    return String(date);
  }
};

const formatKrw = (value) => {
  const num = Number(value);
  if (isNaN(num)) return "₩0";
  return new Intl.NumberFormat("ko-KR", {
    style: "currency",
    currency: "KRW",
  }).format(num); 
};
</script>

<style scoped lang="scss">


@use "/src/assets/style/variables" as *;
// 써머리
.summary_card {
  position: relative;
  width: 100%;
  border-radius: $radius-m;
  background: #fff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  border: 1px solid #f0f0f0;
padding: 15px 40px 10px;
  box-sizing: border-box;
  transition: all 0.3s ease;

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

  &:hover {
    border-color: #d9efeb;
  }

  .card_title {
    font-size: $text-md;
    font-weight: 600;
    color: #222;
    margin: 15px 0;
  }

//요약
  table {
    width: 100%;
    border-collapse: collapse;
    font-size: $text-sm;
    color: #444;
    margin-bottom: 24px;

    tr {
      border-bottom: 1px solid #e7e7e7;

      &:last-child {
        border-bottom: none;
      }

      &.section-title td {
        font-size: $label-md;
        color: #333;
        font-weight: 600;
        padding-top: 12px;
        border-bottom: 1px solid #e7e7e7;
      }

      &.total td {
        font-weight: 600;
        color: #111;

        &:last-child {
          font-size: $text-sm;
          color: $color_main_light;
        }
      }
    }

    td {
      padding: 10px 0;
      text-align: left;
      vertical-align: middle;

      &:first-child {
        color: #777;
        width: 40%;
      }
    }
  }

// 전 안내문
  .summary-guide {
    text-align: center;
    padding: 60px 20px;
    color: #9aa6a9;
    font-size: 15px;
    font-weight: 500;
    background: #f9fbfb;
    border: 1px dashed #cfe2e2;
    border-radius: $radius-m;
    transition: opacity 0.3s ease;
  }

//소제목
  .s-title {
    color: #333 !important;
    font-size: $text-sm !important;
    font-weight: 600;
    margin-top: 12px !important;
  }
}

//제출버튼
.submit_btn {
  width: 70%;
  padding: 14px 0;
  margin: 20px auto 0;
  display: block;
  text-align: center;
  font-weight: 600;
  font-size: $button;
  color: #fff;
  background: $color_main;
  border: none;
  border-radius: $radius-s;
  cursor: pointer;
  transition: 0.3s ease;


}

//반응형
@media (max-width: 1024px) {
  .summary_card {
    width: 90%;
    margin: 0 auto;
    padding: 15px 40px 10px;
  }

  .submit_btn {
    width: 90%;
    font-size: 1rem;
  }
}


</style>
