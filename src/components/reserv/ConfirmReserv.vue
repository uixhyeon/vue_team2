<template>
  <div class="confirm-overlay" @click.self="$emit('close')">
    <div class="confirm-box">
      <h2>입력 내용 확인</h2>

      <!-- 사물함 예약 -->
      <section v-if="form.name || form.phone || form.size || form.address">
        <h3 class="section-title">사물함 예약</h3>
        <ul class="confirm-list">
          <li><strong>이름</strong><span>{{ form.name }}</span></li>
          <li><strong>휴대폰</strong><span>{{ form.phone }}</span></li>
          <li><strong>사물함</strong><span>{{ form.size }}</span></li>
          <li><strong>대여 장소</strong><span>{{ form.address }}</span></li>
          <li v-if="form.dateRange">
            <strong>기간</strong>
            <span>{{ formatDate(form.dateRange[0]) }} ~ {{ formatDate(form.dateRange[1]) }}</span>
          </li>
        </ul>
      </section>

      <!-- 짐 가져오기 -->
      <section v-if="form.pickupAddress">
        <h3 class="section-title">짐 가져오기</h3>
        <ul class="confirm-list">
          <li>
            <strong>픽업 주소</strong>
            <span>{{ form.pickupAddress }} {{ form.pickupAddressDetail }}</span>
          </li>
          <li v-if="form.pickupDate">
            <strong>픽업일</strong>
            <span>{{ formatDate(form.pickupDate) }}</span>
          </li>
        </ul>
      </section>

      <!-- 집으로 보내기 -->
      <section v-if="form.homeAddress">
        <h3 class="section-title">집으로 보내기</h3>
        <ul class="confirm-list">
          <li>
            <strong>배송 주소</strong>
            <span>{{ form.homeAddress }} {{ form.homeAddressDetail }}</span>
          </li>
          <li v-if="form.deliveryDate">
            <strong>배송일</strong>
            <span>{{ formatDate(form.deliveryDate) }}</span>
          </li>
        </ul>
      </section>

      <!-- 최종 금액 -->
      <ul class="confirm-list total">
        <li><strong>예상 금액</strong><span>{{ totalPrice.toLocaleString() }}원</span></li>
      </ul>

      <!-- 버튼 영역 -->
      <div class="confirm-actions">
        <button class="btn line" @click="$emit('close')">수정하기</button>
        <button class="btn primary" @click="$emit('submit')">확인하고 진행</button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  form: { type: Object, required: true },
  totalPrice: { type: Number, default: 0 },
});
defineEmits(["close", "submit"]);

function formatDate(date) {
  if (!date) return "";
  return new Date(date).toLocaleDateString("ko-KR", {
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
  });
}
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;

.confirm-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.35);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.confirm-box {
  background: #fff;
  border-radius: $radius-m;
  padding: 24px 20px;
  width: min(95vw, 420px);

  h2 {
    font-size: 1.15rem;
    font-weight: 700;
    margin-bottom: 1.2rem;
    color: #222;
  }

  .section-title {
    font-size: 0.95rem;
    font-weight: 600;
    margin-bottom: 0.4rem;
    color: $color_main_deep;
    border-left: 4px solid $color_main;
    padding-left: 8px;
  }

  section + section {
    margin-top: 1.2rem;
    padding-top: 1.2rem;
    border-top: 1px solid #e5e5e5;
  }

  .confirm-list {
    list-style: none;
    padding: 0;
    margin-bottom: 0.8rem;
    display: flex;
    flex-direction: column;
    gap: 6px;

    li {
      display: flex;
      justify-content: space-between;
      gap: 8px;
      font-size: 0.9rem;
      color: #333;

      strong {
        font-weight: 500;
        color: #444;
      }

      span {
        text-align: right;
        color: #222;
      }
    }

    &.total {
      margin-top: 1rem;
      border-top: 1px solid #eee;
      padding-top: 0.8rem;

      strong {
        font-weight: 600;
        color: #000;
      }
      span {
        font-weight: 700;
        color: $color_main_deep;
      }
    }
  }

  .confirm-actions {
    display: flex;
    justify-content: flex-end;
    gap: 8px;
    margin-top: 0.8rem;

    .btn {
      border: none;
      border-radius: $radius-s;
      padding: 9px 16px;
      font-size: 0.9rem;
      cursor: pointer;
      transition: background 0.2s ease;
    }

    .btn.line {
      background: #f3f3f3;

      &:hover {
        background: #e7e7e7;
      }
    }

    .btn.primary {
      background: $color_main;
      color: #fff;

      &:hover {
        background: $color_main_deep;
      }
    }
  }
}
</style>