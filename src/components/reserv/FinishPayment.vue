<template>
  <div class="confirm-overlay" @click.self="$emit('close')">
    <div class="confirm-box">
      <!-- 상단 아이콘 -->
      <div class="icon-wrap">
        <i class="fa-solid fa-circle-check"></i>
      </div>

      <!-- 결제 완료 문구 -->
      <h2>결제가 완료되었습니다 🎉</h2>
      <p class="complete-message">
        결제수단: {{ paymentMethod }}<br />
        결제금액: <strong>{{ totalPrice.toLocaleString() }}원</strong><br />
        예약번호: <strong>{{ orderId }}</strong>
      </p>

      <!-- 버튼 -->
  <div class="confirm-actions">
  <button class="btn line" @click="$emit('close')">확인</button>
</div>

    </div>
  </div>
</template>

<script setup>
defineProps({
  totalPrice: { type: Number, default: 0 },
  paymentMethod: { type: String, default: "카드 결제" },
  orderId: { type: String, default: "20251102-0001" },
});

defineEmits(["close", "goHome"]);
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
  padding: 2rem 1.5rem;
  width: min(90vw, 420px);
  text-align: center;
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.1);
  animation: fadeUp 0.35s ease;

  .icon-wrap {
    font-size: 3rem;
    color: $color_main;
    margin-bottom: 1rem;
  }

  h2 {
    font-size: 1.3rem;
    font-weight: 700;
    margin-bottom: 0.6rem;
    color: #222;
  }

  .complete-message {
    background: #f8fdfa;
    border: 1px solid #d5eee9;
    border-radius: $radius-m;
    padding: 14px 16px;
    font-size: 0.95rem;
    line-height: 1.6;
    color: #333;
    margin-bottom: 1.8rem;

    strong {
      color: $color_main_deep;
      font-weight: 600;
    }
  }

  .confirm-actions {
    display: flex;
    justify-content: center;
    gap: 10px;

    .btn {
      width: 120px;
      border: none;
      border-radius: $radius-s ;
      padding: 10px 20px;
      font-weight: 600;
      font-size: 0.95rem;
      cursor: pointer;
      transition: all 0.25s ease;

      &.primary {
        background: $color_main;
        color: #fff;

        &:hover {
          background: $color_main_deep;
        }
      }

      &.line {
        background: #f3f3f3;
        color: #333;

        &:hover {
          background: #e7e7e7;
        }
      }
    }
  }
}

/* 애니메이션 */
@keyframes fadeUp {
  from {
    opacity: 0;
    transform: translateY(15px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
