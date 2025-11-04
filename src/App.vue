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
