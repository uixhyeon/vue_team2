<script setup>
import Header from "./components/Header.vue";
import Footer from "./components/Footer.vue";
import { ref, onMounted, onUnmounted } from "vue";

// 전역 AlertModal을 위한
const showAlert = ref(false);
const alertMessage = ref("");

function onShowAlert(e) {
  alertMessage.value = e?.detail ?? "";
  showAlert.value = true;
}

onMounted(() => {
  window.addEventListener("show-alert", onShowAlert);
  
  // ✅ 카카오맵 SDK 사전 로드 (추가)
  if (!window.kakao?.maps && !document.querySelector('script[src*="kakao.com"]')) {
    const script = document.createElement('script');
    const key = import.meta.env.VITE_KAKAO_MAP_APP_KEY;
    script.src = `https://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=${key}&libraries=services`;
    script.async = true;
    script.onload = () => {
      window.kakao.maps.load(() => {
        console.log('✅ 카카오맵 SDK 사전 로드 완료');
      });
    };
    document.head.appendChild(script);
  }
});

onUnmounted(() => {
  window.removeEventListener("show-alert", onShowAlert);
});
</script>

<template>
  <div class="wrap">
    <Header />
    <main>
      <router-view></router-view>
    </main>
    <Footer />

    <!-- 전역 AlertModal -->
    <AlertModal :show="showAlert" :message="alertMessage" @close="showAlert = false" />
  </div>
</template>

<style scoped></style>