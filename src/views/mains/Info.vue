<!-- InfoPricing.vue (보관 캐러셀 + 배송 2카드 / 배지 제거 / 타이포 통일)
     ▶ 배송 탭 썸네일을 이미지로 교체 (SVG 제거)
     ▶ 넣을 이미지 위치:
        public/images/mains/info/delivery-same.png
        public/images/mains/info/delivery-other.png
-->
<template>
  <div class="info-main">
    <section class="pricing-section" aria-label="요금 안내 섹션">
      <div class="inner">
        <h2 class="title">이용 요금이 궁금하신가요?</h2>

        <!-- 탭 -->
        <div class="tabs" role="tablist">
          <button
            class="tab"
            :class="{ active: activeTab === 'storage' }"
            role="tab"
            :aria-selected="activeTab === 'storage'"
            @click="activeTab = 'storage'">보관</button>
          <button
            class="tab"
            :class="{ active: activeTab === 'delivery' }"
            role="tab"
            :aria-selected="activeTab === 'delivery'"
            @click="activeTab = 'delivery'">배송</button>
        </div>

        <!-- 보관 패널 (기존 캐러셀 유지) -->
        <div v-show="activeTab === 'storage'" class="panel" role="tabpanel" aria-label="보관 요금">
          <div class="carousel">
            <button class="nav prev" aria-label="이전" @click="prev()" :disabled="isAtStart()">‹</button>
            <div class="viewport">
              <div class="track" :style="trackStyle()">
                <article
                  v-for="item in baseStorage"
                  :key="item.id"
                  class="slide"
                  :id="item.id"
                  :style="slideStyle()">
                  <div class="thumb">
                    <img :src="item.img" :alt="item.label" loading="lazy" />
                  </div>
                  <h3 class="label">{{ item.label }}</h3>
                  <p class="meta">{{ item.meta }}</p>
                  <p v-if="item.price" class="price">{{ item.price }}</p>
                </article>
              </div>
            </div>
            <button class="nav next" aria-label="다음" @click="next()" :disabled="isAtEnd()">›</button>
          </div>

          <div class="more-row">
            <router-link class="more" to="/information"> 자세히 보러 가기 → </router-link>
          </div>
        </div>

        <!-- 배송 패널 (2카드, 이미지 버전) -->
        <div v-show="activeTab === 'delivery'" class="panel" role="tabpanel" aria-label="배송 요금">
          <div class="zone-grid">
            <!-- 동일 권역 -->
            <article class="zone-card">
              <div class="zthumb" aria-hidden="true">
                <!-- ✅ 여기 이미지 경로만 바꾸면 됨 -->
<img src="/images/mains/info/deliver-1.png" alt="">              </div>
              <h3 class="ztitle">동일 권역</h3>
              <p class="zdesc">권역 기준 단일 요금</p>
              <p class="zprice">4,000원부터</p>
            </article>

            <!-- 타 권역 -->
            <article class="zone-card">
              <div class="zthumb" aria-hidden="true">
                <!-- ✅ 여기 이미지 경로만 바꾸면 됨 -->
                <img src="/images/mains/info/deliver-2.png" alt="">        

              </div>
              <h3 class="ztitle">타 권역</h3>
              <p class="zdesc">권역 기준 단일 요금</p>
              <p class="zprice">5,500원부터</p>
            </article>
          </div>

          <div class="more-row">
            <router-link class="more" to="/information"> 자세히 보러 가기 → </router-link>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { computed, onMounted, onBeforeUnmount, reactive, ref } from "vue";

/* 보관 카드 */
const baseStorage = [
  { id: "mini",   label: "XS", meta: "소형 가전, 서류, 장난감",  price: "10,000원부터", img: "/images/mains/info/info-xs.png" },
  { id: "small",  label: "S",  meta: "의류 박스, 서랍형 수납",    price: "17,000원부터", img: "/images/mains/info/info-s.png" },
  { id: "medium", label: "M",  meta: "여행 가방, 의류 박스",      price: "23,000원부터", img: "/images/mains/info/info-m.png" },
  { id: "large",  label: "L",  meta: "행거 의류, 중형 가전",      price: "29,000원부터", img: "/images/mains/info/info-l.png" },
  { id: "xlarge", label: "XL", meta: "대형 짐, 스포츠 장비",      price: "35,000원부터", img: "/images/mains/info/info-xl.png" },
];

/* ===== 배송 썸네일: 이미지 경로만 사용 (SVG 제거) =====
   ▶ 아래 두 경로에 이미지를 넣어두면 그대로 렌더링됨.
   ▶ 기본 경로:
      public/images/mains/info/delivery-same.png
      public/images/mains/info/delivery-other.png
   필요하면 경로를 원하는 곳으로 바꿔도 됨. */
const IMG_SAME  = "/images/mains/info/delivery-same.png";
const IMG_OTHER = "/images/mains/info/delivery-other.png";

/* 탭 & (보관)캐러셀 상태 */
const activeTab = ref("storage"); // 필요시 'storage'로 변경
const vw = ref(typeof window !== "undefined" ? window.innerWidth : 1920);
const onResize = () => (vw.value = window.innerWidth);
onMounted(() => window.addEventListener("resize", onResize));
onBeforeUnmount(() => window.removeEventListener("resize", onResize));

const perView = computed(() => (vw.value >= 1280 ? 3 : vw.value >= 769 ? 2 : 1));
const state = reactive({ storage: { index: 0 } });

function next() {
  const total = baseStorage.length;
  const maxIndex = Math.max(0, total - perView.value);
  state.storage.index = Math.min(state.storage.index + 1, maxIndex);
}
function prev() { state.storage.index = Math.max(state.storage.index - 1, 0); }
function isAtStart() { return state.storage.index <= 0; }
function isAtEnd() {
  const total = baseStorage.length;
  const maxIndex = Math.max(0, total - perView.value);
  return state.storage.index >= maxIndex;
}
function trackStyle() {
  const total = baseStorage.length;
  const pct = -((100 / total) * state.storage.index);
  return { width: `${(total * 100) / perView.value}%`, transform: `translateX(${pct}%)`, transition: "transform .45s ease", display: "flex" };
}
function slideStyle() {
  const total = baseStorage.length;
  return { width: `${100 / total}%` };
}
</script>

<style scoped>
.info-main .pricing-section { padding: clamp(24px, 4vw, 56px) 0; color: #111; }
@media (max-width: 768px) { .info-main .pricing-section { padding: 100px 0 0px; } }
@media (max-width: 390px) { .info-main .pricing-section { padding: 100px 0 0px; } }

.info-main .inner { max-width: 1320px; margin: 0 auto; padding: 0 clamp(16px, 4vw, 24px); text-align: center; }
.info-main .title { font-weight: 700; font-size: clamp(26px, 2.8vw, 38px); color: #333; margin-bottom: clamp(20px, 2.5vw, 32px); }

/* 탭 */
.info-main .tabs { display: inline-flex; background: #f9fafa; border-radius: 8px; overflow: hidden; border: 1px solid #d9e1e1; margin: 0 auto clamp(28px, 4vw, 44px); }
@media (max-width: 390px) { .info-main .tabs { margin: 0; } }
.info-main .tab { min-width: 160px; padding: 12px 18px; font-size: clamp(15px, 1.3vw, 20px); color: #888; background: #fff; border: none; cursor: pointer; transition: all 0.25s ease; font-weight: 500; }
.info-main .tab.active { background: #3A8C88; color: #fff; }

/* 보관 캐러셀 */
.info-main .carousel { position: relative; display: grid; grid-template-columns: 48px 1fr 48px; align-items: center; gap: 12px; }
.info-main .nav { width: 48px; height: 48px; border-radius: 50%; border: 1px solid #e1e8e8; background: #fff; font-size: 26px; cursor: pointer; transition: all 0.15s ease; color: #555353;}
.info-main .nav:hover:not(:disabled) { transform: translateY(-2px); box-shadow: 0 4px 12px rgba(0,0,0,0.08); }
.info-main .nav:disabled { opacity: 0.3; cursor: default; }
.info-main .viewport { overflow: hidden; min-height: 280px; }
.info-main .track { will-change: transform; }
.info-main .slide { display:flex; flex-direction:column; align-items:center; justify-content:flex-end; text-align:center; padding: clamp(8px, 1vw, 16px); box-sizing:border-box; }

/* 보관 썸네일 */
.info-main .thumb { width:100%; height:280px; display:flex; justify-content:center; align-items:flex-end; margin-bottom: clamp(10px, 1vw, 14px); overflow:visible; }
.info-main .thumb img { height:100%; max-height:250px; width:auto; object-fit:contain; }

/* 보관 텍스트 */
.info-main .label { font-weight:800; color:#1a8f8d; font-size: clamp(18px, 1.8vw, 28px); margin-bottom:6px; }
.info-main .meta  { color:#222;    font-size: clamp(14px, 1.4vw, 16px); margin-bottom:4px; }
.info-main .price { color:#888;    font-size: clamp(13px, 1.2vw, 15px); }

/* ===== 배송 2카드 (보관과 타이포/간격 동일) ===== */
.zone-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(260px, 1fr));
  gap: clamp(16px, 2vw, 24px);
  align-items: stretch;
  max-width: 760px;
  margin: 0 auto;
}
@media (max-width: 600px) {
  .zone-grid { grid-template-columns: 1fr; }
}
.zone-card {
  border: 1px solid #e2eaea;
  border-radius: 12px;
  padding: 18px 18px 22px;
  background: #fff;
  text-align: center;
  box-shadow: 0 2px 10px rgba(0,0,0,.03);
  margin-top: 30px;
}

/* 배송 썸네일: 보관 썸네일 높이와 동일 */
.zthumb {
  height: 280px;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  margin-bottom: clamp(10px, 1vw, 14px);
}
.zthumb img {
  height: 100%;
  max-height: 250px;
  width: auto;
  object-fit: contain;
}

/* 배송 텍스트 = 보관과 동일한 시스템 */
.ztitle {
  font-weight: 800;
  color: #1a8f8d;
  font-size: clamp(18px, 1.8vw, 28px);
  margin-bottom: 6px;
}
.zdesc {
  color: #222;
  font-size: clamp(14px, 1.4vw, 16px);
  margin-bottom: 4px;
}
.zprice {
  color: #888;
  font-size: clamp(13px, 1.2vw, 15px);
  font-weight: 400;
}

/* 자세히 보기 */
.info-main .more-row { display:flex; justify-content:flex-end; }
.info-main .more { color:#777; font-size: clamp(17px, 1.3vw, 17px); margin-top: 14px; text-decoration: none; }
.info-main .more:hover { text-decoration: underline; }
</style>
