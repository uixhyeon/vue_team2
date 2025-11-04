<template>
  <div
    v-if="open"
    class="addr-modal"
    role="dialog"
    aria-modal="true"
    aria-labelledby="branch-title"
    @click="$emit('close')">
    <div class="dialog" @click.stop>
      <!-- 헤더 -->
      <div class="header">
        <h3 id="branch-title">지점 선택</h3>
        <button class="icon-btn" @click="$emit('close')" aria-label="닫기">✕</button>
      </div>

      <!-- 콘텐츠 -->
      <div class="content">
        <div class="modal-body">
          <!-- 왼쪽: 지점 리스트 -->
          <div class="location-selection-section">
      
      
<div class="location-list">
  <!-- 지역 그룹 -->
  <div v-for="group in locations" :key="group.region" class="region-block">
    <h4 class="region-title">{{ group.region }}</h4>

    <!-- 그룹 내 지점들 -->
    <div
      v-for="location in group.branches"
      :key="location.id"
      class="result-item"
      :class="{
        disabled: location.status === '점검중',
        selected: selectedLocation && selectedLocation.id === location.id,
      }"
      @click="selectLocation(location)"
    >
      <div class="result-info">
        <h5>{{ location.name }}</h5>
        <p>{{ location.address }}</p>
        <p class="locker-info">{{ location.lockers }}</p>
        <div class="location-meta">
          <span class="status" :class="location.status === '운영중' ? 'operating' : 'maintenance'">
            {{ location.status }}
          </span>
        </div>
      </div>
    </div>
  </div>
</div>

          </div>

          <!-- 오른쪽 지도 -->
          <div class="map-section-large">
            <div ref="modalMapEl" class="modal-map-large">
              <!-- 기본 지도 내용 -->
              <div
                style="
                  width: 100%;
                  height: 100%;
                  background: #e8f4f8;
                  border-radius: 8px;
                  display: flex;
                  align-items: center;
                  justify-content: center;
                  position: relative;
                  border: 2px solid #028587;
                ">
                <div
                  style="
                    position: absolute;
                    top: 10px;
                    left: 10px;
                    background: white;
                    padding: 8px;
                    border-radius: 8px;
                    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
                    font-size: 12px;
                  ">
                  📍 지점을 선택해주세요
                </div>
                <div style="font-size: 48px; color: #028587">🗺️</div>
              </div>
            </div>

            <div v-if="selectedLocation" class="location-card">
              <h4>{{ selectedLocation.name }}</h4>
              <p>{{ selectedLocation.address }}</p>
              <p>{{ selectedLocation.lockers }}</p>
              <div class="location-meta">
                <span class="distance">{{ selectedLocation.distance }}</span>
                <span class="status" :class="selectedLocation.status === '운영중' ? 'operating' : 'maintenance'">
                  {{ selectedLocation.status }}
                </span>
              </div>
              <button class="directions-btn" @click="openKakaoMapDirections(selectedLocation)">📍 길찾기</button>
            </div>
          </div>
        </div>
      </div>

      <!-- 푸터 -->
      <div class="footer">
        <button class="btn" @click="confirm" :disabled="!selectedLocation">선택 완료</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, nextTick, onMounted } from "vue";

const props = defineProps({
  open: { type: Boolean, default: false },
  locations: { type: Array, default: () => [] },
});
const emit = defineEmits(["close", "selected"]);

const modalMapEl = ref(null);
const selectedLocation = ref(null);
const mapReady = ref(false)

let map, marker, geocoder;

/* 카카오맵 스크립트 가져오기*/
async function loadKakaoMapScript() {
  if (window.kakao && window.kakao.maps) return;
  const key = import.meta.env.VITE_KAKAO_MAP_APP_KEY;
  await new Promise((resolve) => {
    const s = document.createElement("script");
    s.src = `https://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=${key}&libraries=services`;
    s.onload = () => window.kakao.maps.load(resolve);
    document.head.appendChild(s);
  });
}

/* 지도 마운트 */
async function mountMap() {
  await loadKakaoMapScript();
  const center = new window.kakao.maps.LatLng(35.8714, 128.6014);
  map = new window.kakao.maps.Map(modalMapEl.value, { center, level: 4 });
  marker = new window.kakao.maps.Marker({ position: center });
  marker.setMap(map);
  geocoder = new window.kakao.maps.services.Geocoder();
  window.dispatchEvent(new Event("resize"));
  
  window.kakao.maps.event.trigger(map, "resize"); // ✅ 지도 강제 리렌더링
  console.log("✅ 지도 준비 완료:", map);
    mapReady.value = true;
}

/* 지도 이동 */
function moveMapTo(location) {
  if (!geocoder || !location?.address) return;

  let searchAddress = location.address;

  // 주소 자동 보정
if (!/광역시|특별자치도|도/.test(searchAddress)) {
  if (location.region?.includes("부산")) searchAddress = "부산광역시 " + searchAddress;
  else if (location.region?.includes("강릉") || location.region?.includes("속초"))
    searchAddress = "강원특별자치도 " + searchAddress;
  else if (location.region?.includes("전주"))
    searchAddress = "전라북도 " + searchAddress;
  else if (location.region?.includes("제주"))
    searchAddress = "제주특별자치도 " + searchAddress;
}


  // 오사카 예외
  if (location.region === "오사카") {
    const latlng = new window.kakao.maps.LatLng(34.6695, 135.5008);
    map.setCenter(latlng);
    marker.setPosition(latlng);
    console.log("🌏 오사카 좌표 이동 완료");
    return;
  }

  // 주소 검색
  geocoder.addressSearch(searchAddress, (results, status) => {
    if (status === window.kakao.maps.services.Status.OK && results.length > 0) {
      const { x, y } = results[0];
      const latlng = new window.kakao.maps.LatLng(y, x);
      map.setCenter(latlng);
      marker.setPosition(latlng);
      marker.setMap(map);
      console.log("지도 이동 완료:", searchAddress);
    } else {
      console.warn("주소 검색 실패:", searchAddress, status);
    }
  });
}



/* 지점 선택 */
async function selectLocation(location) {
  if (location.status === "점검중") return;

  const regionGroup = props.locations.find((g) =>
    g.branches.some((b) => b.id === location.id)
  );

  // region을 확실히 포함
  const locWithRegion = { ...location, region: regionGroup?.region || "" };
  
  if (!locWithRegion.region && location.region) {
  locWithRegion.region = location.region; // region 누락 대비
}

  selectedLocation.value = locWithRegion;

  console.log("📍 선택된 지점:", locWithRegion.address);

  // 지도 준비될 때까지 대기
  let tries = 0;
  while (!mapReady.value && tries < 10) {
    console.log("⏳ 지도가 준비 중...");
    await new Promise((r) => setTimeout(r, 200));
    tries++;
  }

  if (mapReady.value && locWithRegion.address) {
    moveMapTo(locWithRegion); // region 포함된 객체로 이동
  } else {
    console.warn("⚠️ 지도 미완성으로 이동 실패");
  }
}


/* 길찾기 */
function openKakaoMapDirections(location) {
  const { name, lat, lng, address } = location;
  const encodedName = encodeURIComponent(name);
  const appUrl = `kakaomap://route?ep=${lat},${lng}&by=FOOT`;
  const webUrl = `https://map.kakao.com/link/to/${encodedName},${lat},${lng}`;
  const isMobile = /Android|iPhone|iPad|iPod/i.test(navigator.userAgent);
  if (isMobile) {
    window.location.href = appUrl;
    setTimeout(() => (window.location.href = webUrl), 800);
  } else {
    window.open(webUrl, "_blank");
  }
}

/* 완료 버튼 */
function confirm() {
  emit("selected", { name: selectedLocation.value.name });
  emit("close");
}
watch(
  () => props.open,
  async (v) => {
    if (v) {
      await loadKakaoMapScript();
      await nextTick();
      if (!map) await mountMap();

setTimeout(() => {
  window.kakao.maps.event.trigger(map, "resize");
  window.dispatchEvent(new Event("resize")); // Safari 보완
  if (selectedLocation.value?.address) {
    moveMapTo(selectedLocation.value);
  }
}, 400);


    }
  }
);


onMounted(async () => {
  if (props.open) {
    await nextTick();
    mountMap();
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
  width: min(900px, 90vw);
  height: min(60vh, 900px);
  background: #fff;
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.25);
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-bottom: 1px solid #eee;
}

.header h3 {
  font-size: 18px;
  font-weight: 700;
  margin: 0;
}

.icon-btn {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  line-height: 1;
}

.content {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.modal-body {
  flex: 1;
  display: flex;
  overflow: hidden;
}

/* 왼쪽 리스트 */
.location-selection-section {
  width: 35%;
  border-right: 1px solid #eee;
  display: flex;
  flex-direction: column;
  padding: 16px;
  overflow-y: auto;
}

.location-list {
  flex: 1;
  overflow-y: auto;
}

.result-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
  transition: background-color 0.2s ease;
}
.result-item:hover {
  background-color: #f8f9fa;
}
.result-item.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
.result-item.selected {
  background: #f0fdfa;
  border-left: 4px solid $color_main;
}

.result-info h4 {
  margin: 0 0 4px 0;
  font-size: 14px;
  font-weight: 600;
}
.result-info p {
  margin: 0;
  font-size: 12px;
  color: #666;
}
.result-icon {
  font-size: 18px;
}

/* 지도 */
.map-section-large {
  width: 65%;
  position: relative;
}
.modal-map-large {
  width: 100%;
  height: 100%;
  min-height: 500px;
  border-radius: 8px;
  border: 1px solid #ddd;
  overflow: hidden;
}

/* 지도 카드 */
.location-card {
  position: absolute;
  bottom: 16px;
  right: 16px;
  background: #fff;
  padding: 12px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  max-width: 240px;
}
.location-card h4 {
  margin: 0 0 4px;
  font-size: 14px;
  font-weight: 600;
}
.location-card p {
  margin: 0 0 4px;
  font-size: 12px;
  color: #666;
}
.location-card .location-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 11px;
  margin-top: 2px;
}
.location-card .distance {
  color: #028587;
  font-weight: 600;
}
.location-card .status.operating {
  background: #e8f5e8;
  color: #2d5a2d;
  padding: 2px 6px;
  border-radius: 12px;
  font-weight: 600;
  font-size: 10px;
}
.location-card .status.maintenance {
  background: #ffe8e8;
  color: #8b0000;
  padding: 2px 6px;
  border-radius: 12px;
  font-weight: 600;
  font-size: 10px;
}
.directions-btn {
  width: 100%;
  background: $color_main;
  color: #fff;
  border: none;
  padding: 6px 12px;
  border-radius: 8px;
  font-size: 12px;
  cursor: pointer;
  margin-top: 6px;
}

/* 푸터 */
.footer {
  padding: 12px 16px;
  border-top: 1px solid #f0f0f0;
}
.footer .btn {
  width: 100%;
  padding: 12px;
  background: $color_main;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  font-size: 16px;
  cursor: pointer;
  transition: 0.2s ease;
}
.footer .btn:hover {
  background: $color_main_deep;
}
.footer .btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 반응형 */
@media (max-width: 1024px) {
  .dialog {
    width: 95vw;
    height: 85vh;
  }
  .location-selection-section {
    width: 40%;
  }
  .map-section-large {
    width: 60%;
  }
}
@media (max-width: 768px) {
  .dialog {
    width: 94vw;
    height: 90vh;
  }
  .modal-body {
    flex-direction: column;
  }
  .location-selection-section {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid #eee;
    padding: 12px;
  }
  .map-section-large {
    width: 100%;
    height: 55%;
  }
  .modal-map-large {
    min-height: 300px;
  }
  .location-card {
    position: relative;
    bottom: auto;
    right: auto;
    margin: 12px;
    max-width: none;
  }
}
@media (max-width: 480px) {
  .dialog {
    width: 90vw;
    height: 85vh;
  }
  .result-item {
    padding: 10px 12px;
  }
  .result-info h4 {
    font-size: 13px;
  }
  .result-info p {
    font-size: 11px;
  }
}

// 추가
.region-block {
  margin-bottom: 1.2rem;
}

.region-title {
  font-size: 15px;
  font-weight: 700;
  color: $color_sub ;
  margin-bottom: 0.5rem;
  border-left: 4px solid $color_sub_deep ;
  padding-left: 8px;
}

</style>
