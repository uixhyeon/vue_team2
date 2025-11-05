<!-- NavBar.vue -->
<template>
  <nav class="jb-nav" :class="{ 'is-open': isOpen }">
    <!-- Left: Logo -->
    <router-link
      to="/"
      class="logo"
      aria-label="홈으로 이동"
      @click.native="scrollToTop"
    >
      <img class="logo-img" src="/images/mains/header/logo-1.png" alt="마타주 로고" />
    </router-link>

    <!-- Desktop Menu -->
    <div class="menu">
      <router-link class="dropdown" to="/information">이용안내</router-link>
      <router-link class="dropdown" to="/login">예약하기
      </router-link>
      <router-link to="/promotion">프로모션</router-link>
      <router-link class="dropdown" to="/community">커뮤니티</router-link>
      <router-link class="dropdown" to="/support">고객센터</router-link>
      <div class="login">
        <router-link to="/login">로그인</router-link>
        <router-link to="/signup">예약 확인</router-link>
      </div>
    </div>

    <!-- Mobile Right -->
    <div class="nav-right">
      <div class="login-mini">
        <router-link to="/login">로그인</router-link>
        <router-link to="/findreserv">예약 확인</router-link>
      </div>

      <button
        class="hamburger"
        :aria-expanded="isOpen ? 'true' : 'false'"
        aria-controls="mobile-panel"
        @click="toggle()"
      >
        <span class="bar" />
        <span class="bar" />
        <span class="bar" />
        <span class="sr-only">메뉴 열기</span>
      </button>
    </div>
  </nav>

  <div class="nav-spacer" aria-hidden="true"></div>

  <transition name="slide">
    <aside v-show="isOpen" id="mobile-panel" class="mobile-panel" @click.self="close()">
      <div class="panel-inner">
        <button class="panel-close" @click="close()" aria-label="메뉴 닫기">×</button>

        <div class="quick-row">
          <router-link to="/login" class="quick-link" @click="close()">로그인</router-link>
          <span class="sep" aria-hidden="true">|</span>
          <router-link to="/findreserv" class="quick-link" @click="close()">예약 확인</router-link>
        </div>

        <ul class="mobile-list">
          <li><router-link to="/information" class="m-title link-title" @click="close()">이용안내</router-link></li>
          <li><router-link to="/login" class="m-title link-title" @click="close()">예약하기</router-link></li>
          <li><router-link to="/promotion" class="m-title link-title" @click="close()">프로모션</router-link></li>
          <li><router-link to="/community" class="m-title link-title" @click="close()">커뮤니티</router-link></li>
          <li><router-link to="/support" class="m-title link-title" @click="close()">고객센터</router-link></li>
        </ul>
      </div>
    </aside>
  </transition>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch, reactive } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

const isOpen = ref(false)
const toggle = () => (isOpen.value = !isOpen.value)
const close = () => (isOpen.value = false)

const scrollToTop = async (e) => {
  if (route.path === '/') {
    e.preventDefault()
    window.scrollTo({ top: 0, behavior: 'smooth' })
  } else {
    await router.push('/')
  }
}

const section = reactive({ guide: false, reserve: false, support: false })
const toggleSection = (key) => (section[key] = !section[key])

const onEsc = (e) => { if (e.key === 'Escape') close() }
onMounted(() => window.addEventListener('keydown', onEsc))
onBeforeUnmount(() => {
  window.removeEventListener('keydown', onEsc)
  document.documentElement.style.overflow = ''
  document.body.style.overflow = ''
})
watch(isOpen, (open) => {
  document.documentElement.style.overflow = open ? 'hidden' : ''
  document.body.style.overflow = open ? 'hidden' : ''
})
</script>

<style scoped lang="scss">
@use "/src/assets/style/variables" as *;
:root { --nav-h: 76px; }
@media (max-width: 768px){ :root { --nav-h: 64px; } }

.jb-nav {
  position: fixed;
  top: 0; left: 0; right: 0;
  z-index: 9999;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: clamp(8px, 1vw, 20px);
  padding: 10px min(8vw, 100px);
  border-bottom: 1px solid #e7e2e2;
  line-height: 60px;
  background: #fff;
  min-height: var(--nav-h);
}
.nav-spacer{ height: var(--nav-h); }

.logo {
  width: auto;
  font-size: clamp(30px, 3vw, 38px);
  font-weight: 600;
  color: $color_main;
  text-decoration: none;
  display: inline-flex;
  align-items: center;
  line-height: 1;
}
.logo-img {
  height: 1.7em;
  width: auto;
  display: block;
  object-fit: contain;
}

.menu {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: clamp(15px, 2vw, 30px);
  font-size: clamp(15px, 1vw, 20px);
}

.menu > a,
.menu > .dropdown {
  color: #333;
  font-weight: 700;
  text-decoration: none;
  padding: 5px 10px;
  position: relative;
  font-size: clamp(15px, 2vw, 22px);
}
.menu > a:hover,
.menu > .dropdown:hover { color: $color_main_deep; }

/* ✅ 클릭(현재 라우트)일 때도 동일한 색 */
.menu > a.router-link-active,
.menu > a.router-link-exact-active,
.menu > .dropdown.router-link-active,
.menu > .dropdown.router-link-exact-active {
  color: $color_main_deep;
}

/* ✅ 로그인/예약확인 버튼도 동일하게 호버 + active 색상 */
.login a {
  color: #a0a0a0;
  font-size: clamp(12px, 1vw, 16px);
  text-decoration: none;
  transition: color 0.2s;
}
.login a:hover,
.login a.router-link-active,
.login a.router-link-exact-active {
  color: $color_main_deep;
  font-weight: 600;
}

.login { padding-left: 2vw; }

/* ===== Dropdown ===== */
.dropdown { position: relative; display: inline-flex; align-items: center; }
.dropdown .submenu {
  display: none;
  position: absolute;
  top: 100%; left: 50%;
  transform: translateX(-50%);
  margin-top: -2px;
  padding: 8px 0;
  border-radius: 10px;
  min-width: 160px;
  background: rgba(255,255,255,0.9);
  text-align: center;
  transition: all 0.2s ease;
  z-index: 10001;
}
.dropdown:hover .submenu { display: block; }

.submenu li a {
  display: block;
  padding: 12px 24px;
  color: #333;
  text-decoration: none;
  font-size: 16px;
}
.submenu li a:hover { color: $color_main_deep; }

.nav-right { display: none; align-items: center; gap: 14px; }
.login-mini { display: flex; align-items: center; gap: 12px; }
.login-mini a { color: #6f6f6f; font-size: 16px !important; text-decoration: none; transition: color 0.2s; }
.login-mini a:hover,
.login-mini a.router-link-active,
.login-mini a.router-link-exact-active {
  color: $color_main_deep;
  font-weight: 600;
}

/* Hamburger */
.hamburger {
  width: 34px; height: 28px; display: inline-flex;
  flex-direction: column; justify-content: space-between; align-items: center;
  border: 0; background: transparent; cursor: pointer; padding: 2px;
}
.hamburger .bar {
  width: 100%; height: 3px;
  background: #444; border-radius: 2px;
  transition: transform .25s ease, opacity .25s ease;
}
.jb-nav.is-open .hamburger .bar:nth-child(1){ transform: translateY(9px) rotate(45deg); }
.jb-nav.is-open .hamburger .bar:nth-child(2){ opacity: 0; }
.jb-nav.is-open .hamburger .bar:nth-child(3){ transform: translateY(-9px) rotate(-45deg); }

.mobile-panel {
  position: fixed; inset: 0;
  background: rgba(0,0,0,.35);
  z-index: 12000;
}
.panel-inner {
  position: absolute; right: 0;
  top: var(--nav-h);
  width: min(78vw, 360px);
  height: calc(100% - var(--nav-h));
  background: #fff;
  box-shadow: -6px 0 16px rgba(0,0,0,.08);
  padding: 28px 22px;
  overflow-y: auto;
}
.panel-close {
  position: absolute; top: 8px; right: 10px;
  font-size: 28px;
  background: transparent; border: none; cursor: pointer; color: #333;
}

.mobile-list { margin-top: 24px; display: flex; flex-direction: column; gap: 16px; }
.m-title.link-title {
  font-size: 20px; font-weight: 700; color: $color_main; text-decoration: none;
}

@media (max-width: 1000px) {
  .menu { gap: 10px; }
  .login { padding-left: 0; }
}
@media (max-width: 768px) {
  .menu { display: none; }
  .nav-right { display: flex; }
  .login-mini { display: none; }
}
@media (max-width: 390px) {
  .login-mini { gap: 10px; }
  .login-mini a { font-size: 13px; }
}

.sr-only {
  position: absolute !important;
  width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0;
}

@media (min-width: 769px) {
  .nav-right { display: flex; align-items: center; gap: 14px; }
  .hamburger { display: none; }
  .login { display: none; }
  .menu { margin-left: auto; margin-right: 2vw; justify-content: flex-end; }
}

.quick-row{
  display:flex; align-items:center; gap:12px;
  margin-top:8px; margin-bottom:18px;
}
.quick-link{
  font-size:16px; font-weight:700; color:#1f1f1f;
  text-decoration:none; padding:4px 2px; transition:color .2s;
}
.quick-link:hover,
.quick-link.router-link-active,
.quick-link.router-link-exact-active {
  color: $color_main_deep;
  text-decoration:underline;
}
.sep{ color:#d1d5db; }

.jb-nav{
  position: sticky;
  top: 0;
  left: 0; right: 0;
  z-index: 9999;
}
.mobile-panel{ position: fixed; inset: 0; height: 100dvh; }
.panel-inner{ position: absolute; top: 0; right: 0; bottom: 0; overflow-y: auto; }
</style>
