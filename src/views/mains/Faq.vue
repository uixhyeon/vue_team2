<template>
  <div class="faq-main">
    <section class="download-faq">
      <div class="inner">
        <!-- 왼쪽: 앱 다운로드 -->
        <article class="download-card" ref="cardRef" aria-label="앱 다운로드">
          <svg class="bg-svg" viewBox="0 0 1200 600" preserveAspectRatio="none" aria-hidden="true">
            <defs>
              <linearGradient id="mintGrad" x1="0" y1="0" x2="1" y2="0">
                <stop offset="0" stop-color="#2EB5B2"/>
                <stop offset="1" stop-color="#179A99"/>
              </linearGradient>
            </defs>
            <rect width="1200" height="600" fill="url(#mintGrad)" />
            <path d="M0,140 C240,60 420,60 600,160 C820,280 980,260 1200,190 L1200,0 L0,0 Z"
                  fill="#147e7f" fill-opacity="0.35"/>
            <path d="M0,330 C250,280 430,300 620,350 C820,405 980,390 1200,350 L1200,600 L0,600 Z"
                  fill="#ffffff" fill-opacity="0.12"/>
          </svg>

          <div class="card-inner">
            <div class="phone-mock" aria-hidden="true">
              <div class="notch"></div>
              <div class="screen">
                <!-- ✅ 여기: 텍스트 대신 로고 이미지 들어갈 자리 -->
                <img
                  class="screen-logo"
                  src="/images/mains/header/logo-1.png"
                  alt="마타주 로고"
                />
              </div>
            </div>

            <div class="copy">
              <h2 class="title"><strong>마타주</strong> 앱 <span>다운받기</span></h2>
              <p class="desc">
                심플하게, 가볍게 지금 경험해 보세요
              </p>

              <div class="store-btns">
                <a class="store-btn store-btn--white" href="#">
                  <span class="store-left">
                    <span class="icon">📱</span>
                    <span class="store-text">Google Play</span>
                  </span>
                  <span class="store-badge">안드로이드</span>
                </a>
                <a class="store-btn store-btn--white" href="#">
                  <span class="store-left">
                    <span class="icon">🍎</span>
                    <span class="store-text">App Store</span>
                  </span>
                  <span class="store-badge">IOS</span>
                </a>
              </div>
            </div>
          </div>
        </article>

        <!-- 오른쪽: FAQ (높이 고정 + 내부 스크롤) -->
        <aside class="faq" ref="faqRef" aria-label="자주 묻는 질문">
          <header class="faq-head">
            <h3 class="faq-title">자주 묻는 질문</h3>
            <!-- <button type="button" class="inquiry-btn">문의하기</button> -->
          </header>

          <ul class="faq-list" ref="listRef">
            <li v-for="(item, i) in faqs" :key="i" class="faq-item">
              <button class="faq-q" :aria-expanded="openIndex === i" @click="toggle(i)">
                <span class="q-prefix">Q.</span>
                <span class="q-text">{{ item.q }}</span>
                <span class="chev" :class="{ open: openIndex === i }" aria-hidden="true"></span>
              </button>

              <div class="faq-a-wrap" :class="{ open: openIndex === i }">
                <div class="faq-a">{{ item.a }}</div>
              </div>
            </li>
          </ul>
        </aside>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, nextTick, watch } from 'vue'

const openIndex = ref(-1)
const cardRef = ref(null)
const faqRef  = ref(null)
const listRef = ref(null)

const faqs = [
  { q: '보관 가능한 짐의 종류에는 어떤 것이 있나요?', a: '일반 여행가방, 백팩, 유모차, 골프백 등 대부분 보관 가능하며, 위험물·가연성 물질·현금/귀금속은 제한됩니다.' },
  { q: '짐을 맡기고 중간에 꺼낼 수 있나요?', a: '보관 중에도 앱에서 일시 출입을 신청하면 꺼낼 수 있어요. 지점 정책에 따라 추가 확인이 있을 수 있습니다.' },
  { q: '짐을 맡길 때 꼭 신분증이 필요한가요?', a: '첫 보관 시 본인 인증이 필요합니다. 앱에서 간편 인증으로 처리하며, 현장 확인이 필요한 경우 안내드립니다.' },
  { q: '보관 중 분실이나 파손이 생기면 어떻게 보상되나요?', a: '안심 보상 정책을 운영합니다. 문제 발생 시 앱 문의하기로 접수하면 확인 후 약관 범위 내에서 보상합니다.' },
  { q: '짐을 해외로도 보낼 수 있나요?', a: '국내 배송을 기본으로 하고, 일부 지점은 국제 배송 중개를 지원합니다. 가능 지점은 앱에서 확인해 주세요.' },
  { q: '배송 시간을 지정할 수 있나요?', a: '희망 시간대를 선택할 수 있으며, 지역/지점 사정에 따라 조정될 수 있습니다.' }
]

/* ----- FAQ 아코디언: height 0 ↔︎ scrollHeight 애니메이션 ----- */
function openPanel(el) {
  el.style.willChange = 'height'
  el.style.height = '0px'
  requestAnimationFrame(() => {
    el.style.height = el.scrollHeight + 'px'
  })
  el.addEventListener('transitionend', function onEnd(e) {
    if (e.propertyName === 'height') {
      el.style.height = 'auto'
      el.style.willChange = ''
      el.removeEventListener('transitionend', onEnd)
    }
  })
}
function closePanel(el) {
  el.style.willChange = 'height'
  const cur = el.scrollHeight
  el.style.height = cur + 'px'
  requestAnimationFrame(() => {
    el.style.height = '0px'
  })
  el.addEventListener('transitionend', function onEnd(e) {
    if (e.propertyName === 'height') {
      el.style.willChange = ''
      el.removeEventListener('transitionend', onEnd)
    }
  })
}

const toggle = async (i) => {
  const prev = openIndex.value
  openIndex.value = prev === i ? -1 : i
  await nextTick()
  const wraps = faqRef.value.querySelectorAll('.faq-a-wrap')
  wraps.forEach((wrap, idx) => {
    if (openIndex.value === idx) {
      if (wrap.style.height !== 'auto') openPanel(wrap)
    } else {
      if (wrap.style.height !== '0px') closePanel(wrap)
    }
  })
}

/* ----- 좌/우 높이 락: 데스크톱에서만 동일 높이로 고정 ----- */
const lockHeights = () => {
  if (!cardRef.value || !faqRef.value) return
  const w = window.innerWidth
  if (w <= 1000) {
    // 모바일: 자동
    cardRef.value.style.height = ''
    faqRef.value.style.height  = ''
    if (listRef.value) listRef.value.style.overflow = ''
    return
  }
  // 좌측 실제 콘텐츠 기준으로 높이 고정
  const inner = cardRef.value.querySelector('.card-inner')
  const H = inner ? inner.offsetHeight : cardRef.value.offsetHeight
  cardRef.value.style.height = H + 'px'
  faqRef.value.style.height  = H + 'px'
  // 오른쪽은 헤더를 제외한 리스트만 스크롤
  if (listRef.value) listRef.value.style.overflow = 'auto'
}

const onResize = () => {
  // 리플로우 안정화를 위해 한 프레임 뒤 계산
  requestAnimationFrame(lockHeights)
}

onMounted(async () => {
  await nextTick()
  // 초기: 모든 답변 닫기
  faqRef.value.querySelectorAll('.faq-a-wrap').forEach(w => (w.style.height = '0px'))
  // 폰트/이미지 로딩 오차 대비해서 두 번 락
  lockHeights()
  setTimeout(lockHeights, 60)
  window.addEventListener('resize', onResize)
})

watch(openIndex, async () => {
  await nextTick()
  // 토글해도 좌/우 높이는 그대로 유지 (리스트 내부만 스크롤)
  requestAnimationFrame(lockHeights)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', onResize)
})
</script>

<style scoped>
/* 가로 흔들림 방지 */
/* :global(html){ scrollbar-gutter: stable both-edges; } */

/* ===== 변수 ===== */
.faq-main {
  --mint-1: #2EB5B2;
  --mint-2: #179A99;
  --mint-dark: #147e7f;
  --line: #e6eeee;
}

/* ===== 레이아웃 ===== */
.faq-main .download-faq { padding: 32px 16px; }
.faq-main .inner {
  max-width: 1320px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 32px;
  align-items: stretch;
}

/* ===== 왼쪽 카드 ===== */
.faq-main .download-card {
  position: relative;
  overflow: hidden;
  border-radius: 12px;
  padding: 0;
  background: transparent;
  box-shadow: none;
  height: auto;
}
.faq-main .bg-svg { position:absolute; inset:0; width:100%; height:100%; display:block; }

.faq-main .card-inner{
  position: relative; z-index: 1; display: grid;
  grid-template-columns: 260px 1fr; align-items: center;
  gap: 24px; padding: 32px 24px; height: 100%; box-sizing: border-box;
}

/* 폰 */
.faq-main .phone-mock{ width:160px; height:320px; border-radius:20px; background:#fff; position:relative; border:2px solid #0f585a;}
.faq-main .phone-mock .notch{ position:absolute; top:12px; left:50%; transform:translateX(-50%); width:80px; height:18px; border-bottom-left-radius:8px; border-bottom-right-radius:10px; background:#0f585a;}
.faq-main .phone-mock .screen{
  position:absolute; inset:18px; border-radius:20px; background:#ffffff;
  display:flex; align-items:center; justify-content:center;
}
.faq-main .screen-brand{
  display:none;
}
/* ✅ 로고 이미지 자리 (기존 텍스트 크기 느낌 유지) */
.faq-main .screen-logo{
  max-width: 110px;
  width: 100%;
  height: auto;
  object-fit: contain;
  display: block;
}

/* 텍스트 */
.faq-main .copy .title{ color:#ffffff; font-size:28px; font-weight:900; line-height:1.2; margin:0 0 10px;}
.faq-main .copy .desc{ color:#eaf6f6; font-size:14px; line-height:1.6; margin:0 0 18px;}

/* 스토어 버튼 */
.faq-main .store-btns{ display:grid; gap:10px; max-width:320px;}
.faq-main .store-btn{ display:flex; align-items:center; justify-content:space-between; height:46px; padding:0 12px 0 10px; border-radius:10px; text-decoration:none;}
.faq-main .store-btn--white{ background:#fff; color:#1b1b1b; border:1px solid #dfe9e9; box-shadow:0 1px 0 rgba(0,0,0,.04), 0 6px 18px rgba(0,0,0,.05);}
.faq-main .store-left{ display:flex; align-items:center; gap:8px;}
.faq-main .store-text{ font-weight:700; font-size:14px;}
.faq-main .store-badge{ font-size:12px; font-weight:800; background:#eef6f6; color:#0f5f61; padding:6px 12px; border-radius:999px; border:1px solid #d2e8e8;}

/* ===== 오른쪽 FAQ (고정 높이 + 내부 스크롤) ===== */
.faq-main .faq{
  display:flex; flex-direction:column;
  background:#fff; border-radius:8px;
  /* 높이는 JS로 lockHeights()에서 설정 */
}
.faq-main .faq-head{ display:flex; align-items:center; justify-content:space-between; padding:16px 14px 8px; }
.faq-main .faq-title{ font-size:28px; font-weight:900; color:#111; margin-top: -3px;}
.faq-main .inquiry-btn{ height:32px; padding:0 14px; border-radius:999px; border:1px solid #d8e2e2; background:#fff; color:#2b2b2b; font-size:13px; font-weight:600;}
.faq-main .inquiry-btn:hover{ background:#f7fafa; }

.faq-main .faq-list{
  list-style:none; margin:0; padding:0 0 8px 0;
  flex:1;            /* 헤더 제외 영역 채우기 */
  overflow:auto;     /* 내부 스크롤 */
  background:#fff;
}
.faq-main .faq-item{ border-top:1px solid #e9ecec; }
.faq-main .faq-item:first-child{ border-top:none; }

.faq-main .faq-q{
  width:100%; display:flex; align-items:center; gap:10px;
  padding:16px 14px; background:#fff; border:0; text-align:left; cursor:pointer;
  font-size:15px; font-weight:700; color:#222; transition:background .2s ease;
}
.faq-main .faq-q:hover{ background:#fafafa; }
.faq-main .q-prefix{ color:#6f7b7b; font-weight:700; }
.faq-main .q-text{ flex:1; white-space:normal; word-break:keep-all; }

.faq-main .faq-a-wrap{
  height:0; overflow:hidden; transition:height .28s ease;
}
.faq-main .faq-a{
  padding:12px 14px 16px 36px; color:#444; font-size:14px; line-height:1.7;
  white-space:normal; word-break:keep-all;
}

/* 화살표 */
.faq-main .chev{ position:relative; width:16px; height:16px; display:inline-block;}
.faq-main .chev::before, .faq-main .chev::after{
  content:""; position:absolute; top:6px; width:8px; height:2px; background:#666; border-radius:1px; transition:transform .25s ease;
}
.faq-main .chev::before{ left:2px; transform:rotate(45deg); }
.faq-main .chev::after{ right:2px; transform:rotate(-45deg); }
.faq-main .chev.open::before{ transform:rotate(-45deg); }
.faq-main .chev.open::after{ transform:rotate(45deg); }

/* ====== Responsive: 1000px 이하에서 세로 스택 ====== */
@media (max-width: 1000px){
  .faq-main .inner{ grid-template-columns:1fr; gap:24px; align-items:stretch; max-width:700px; }
  .faq-main .download-card{ height:auto!important; }
  .faq-main .card-inner{ grid-template-columns:1fr; gap:16px; padding:24px 18px; }
  .faq-main .phone-mock{ margin:0 auto; width:140px; height:280px; }
  .faq-main .screen-brand{ font-size:18px; }
  .faq-main .copy{ text-align:center; }
  .faq-main .copy .title{ font-size:24px; margin-bottom:8px; }
  .faq-main .copy .desc{ font-size:13px; margin-bottom:14px; }
  .faq-main .store-btns{ max-width:100%; }
  .faq-main .store-btn{ height:44px; }

  .faq-main .faq{ height:auto!important; } /* 모바일은 자동 높이 */
  .faq-main .faq-head{ padding:8px 14px; }
  .faq-main .faq-title{ font-size:24px; }
  .faq-main .faq-q{ padding:14px 12px; font-size:14px; }
  .faq-main .faq-a{ padding:10px 12px 14px 32px; font-size:13px; }
}

/* ====== 390px 이하에서 컴팩트 ====== */
@media (max-width: 390px){
  .faq-main .download-faq{ padding:24px 12px; }
  .faq-main .card-inner{ padding:20px 14px; gap:12px; }
  .faq-main .phone-mock{ width:120px; height:240px; }
  .faq-main .screen-brand{ font-size:16px; }
  .faq-main .copy .title{ font-size:20px; }
  .faq-main .copy .desc{ font-size:12px; }
  .faq-main .store-btn{ height:42px; padding:0 10px; }
  .faq-main .store-text{ font-size:13px; }
  .faq-main .store-badge{ font-size:11px; padding:5px 10px; }
  .faq-main .faq-title{ font-size:20px; }
  .faq-main .faq-q{ padding:12px 10px; font-size:13px; }
  .faq-main .faq-a{ padding:10px 10px 14px 28px; font-size:12px; }
}
</style>
