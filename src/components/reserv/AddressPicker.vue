<template>
  <div
    v-if="open"
    class="addr-modal"
    role="dialog"
    aria-modal="true"
    aria-labelledby="addr-title"
  >
    <div class="dialog">
      <div class="header">
        <h3 id="addr-title">주소 선택</h3>
        <button class="icon-btn" @click="$emit('close')" aria-label="닫기">
          ✕
        </button>
      </div>

      <!-- 지도/주소 검색 -->
      <div class="content">
        <div ref="postcodeWrap" class="postcode-wrap"></div>
        <p class="hint">선택을 하시면 주소가 자동 입력됩니다.</p>

        <div v-if="hasKakaoKey" class="map-wrap">
          <div ref="mapEl" class="map"></div>
          <p class="hint">지도를 드래그해 위치를 조정할 수 있습니다.</p>
        </div>
      </div>

      <div class="footer">
        <button class="btn" @click="confirm" :disabled="!localAddress">
          확인
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, nextTick } from "vue";

const props = defineProps({
  modelValue: { type: String, default: "" },
  open: { type: Boolean, default: false },
});
const emit = defineEmits(["update:modelValue", "close", "selected"]);

const localAddress = ref("");
const postcodeWrap = ref(null);
const mapEl = ref(null);
const hasKakaoKey = Boolean(import.meta.env.VITE_KAKAO_MAP_APP_KEY);
let map, marker, geocoder;

// 주소 선택 완료 시 실행
function confirm() {
  if (!localAddress.value) return;
  emit("update:modelValue", localAddress.value);
  emit("selected", localAddress.value);
  setTimeout(() => emit("close"), 400); //  0.4초 대기
}

// 모달 열릴 때마다 주소창 새로 검색
watch(
  () => props.open,
  async (v) => {
    if (v) {
      localAddress.value = props.modelValue || "";
      await nextTick(); // DOM 렌더 후 실행 되게
      mountPostcode();
    }
  }
);

// 주소 API 가져오기
function loadScript(src) {
  return new Promise((resolve, reject) => {
    if (document.querySelector(`script[src="${src}"]`)) return resolve();
    const s = document.createElement("script");
    s.src = src.startsWith("http") ? src : `https:${src}`;
    s.onload = () => resolve();
    s.onerror = reject;
    document.head.appendChild(s);
  });
}

async function mountPostcode() {
  await loadScript(
    "//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js"
  );
  const Postcode = new window.daum.Postcode({

oncomplete: (data) => {
  //  한글 주소만 저장하도록 수정
  const addr = data.roadAddress || data.jibunAddress || data.address;

  localAddress.value = addr;
  emit("update:modelValue", addr);
  emit("selected", addr);

  setTimeout(() => emit("close"), 800); // 주소 선택 후 자동 닫힘

  if (hasKakaoKey) moveMapTo(addr);
},

    width: "100%",
    height: "420px",
  });
  Postcode.embed(postcodeWrap.value);

  // Kakao 지도 선택적으로 표시
  if (hasKakaoKey) {
    const key = import.meta.env.VITE_KAKAO_MAP_APP_KEY;
    await loadScript(
      `https://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=${key}&libraries=services`
    );
    window.kakao.maps.load(() => {
      geocoder = new window.kakao.maps.services.Geocoder();
      map = new window.kakao.maps.Map(mapEl.value, {
        center: new window.kakao.maps.LatLng(37.4979, 127.0276),
        level: 4,
      });
      marker = new window.kakao.maps.Marker({ position: map.getCenter() });
      marker.setMap(map);
      window.kakao.maps.event.addListener(map, "dragend", () => {
        marker.setPosition(map.getCenter());
      });
      if (localAddress.value) moveMapTo(localAddress.value);
    });
  }
}

function moveMapTo(address) {
  if (!geocoder || !map) return;
  geocoder.addressSearch(address, (results, status) => {
    if (status === window.kakao.maps.services.Status.OK) {
      const { x, y } = results[0];
      const latlng = new window.kakao.maps.LatLng(y, x);
      map.setCenter(latlng);
      marker.setPosition(latlng);
    }
  });
}

onMounted(async () => {
  if (props.open) {
    await nextTick();
    mountPostcode();
  }
});
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;
.addr-modal {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding-top: 5%;
}
.dialog {
  width: min(720px, 80vw);
  background: #fff;

  border-radius: $radius-m;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  // ====================================
  @media (max-width: 768px) {
    min-width: 330px;
  }
}
//================카카오안의 글자조정=============================
.post_search .placeholder {
  color: #969696;
  position: absolute;
  top: 13px;
  left: 20px;
  font-size: 13px;
  cursor: text;
  display: none;
  font-family: dotum, "돋움", sans-serif;
  line-height: 22px;
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  border-bottom: 1px solid #eee;
}
.icon-btn {
  background: transparent;
  border: none;
  font-size: 18px;
  cursor: pointer;
}
.content {
  padding: 12px;
  @media (max-width: 480px) {
    height: 300px;
  }
}
.postcode-wrap {
  width: 100%;

  border: 1px solid #eee;
  border-radius: $radius-s;
  overflow: hidden;
  position: relative;
  @media (max-width: 480px) {
    height: 100%;
  }
}

/* 선택완료 */
.postcode-wrap::before {
  content: "✔️ 선택 완료";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: $color_main;
  font-weight: 600;
  font-size: 18px;
  opacity: 0.85;
  text-align: center;
  letter-spacing: 0.5px;
  pointer-events: none;
  animation: fadeIn 0.2s ease;
}
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translate(-50%, -45%);
  }
  to {
    opacity: 0.85;
    transform: translate(-50%, -50%);
  }
}
.hint {
  color: #888;
  font-size: 12px;
  margin: 8px 2px;
}
.footer {
  padding: 12px;
  border-top: 1px solid #f0f0f0;
}
.btn {
  width: 100%;
  padding: 12px;
  background: $color_main;
  color: #fff;
  border: none;
  border-radius: $radius-s;
  font-size: 16px;
}
/* 비활성화 */
.footer {
  padding: 12px;
  border-top: 1px solid #f0f0f0;
}

.btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: $radius-s;
  font-size: 16px;
  cursor: pointer;
  transition: 0.25s;
}

/* 선택전 비활성화 상태  */
.btn:disabled {
  background: #d1d1d1;
  color: #fff;
  cursor: not-allowed;
  box-shadow: none;
}

/* 선택후 활성화 상태  */
.btn:not(:disabled) {
  background: $color_main;
  color: #fff;
  box-shadow: 0 2px 6px rgba(44, 163, 159, 0.3);
}

.btn:not(:disabled):hover {
  background: $color_main_deep;
}
@media (max-width: 480px) {
  .daum_popup {
    height: 200px !important;
  }
}
</style>
