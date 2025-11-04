<template>
  <div class="search-dock">
    <div class="search-bar" role="search" aria-label="보관/배송 검색" ref="dockRef">
      <!-- 목적지 -->
      <div class="search-item" :class="{ active: openPanel === 'dest' }" @click="toggle('dest')">
        <label class="label">목적지 찾기</label>
        <input
          type="text"
          :value="destination || ''"
          :placeholder="destination ? '' : '어느 지역을 방문하시나요?'"
          readonly />
        <div v-if="openPanel === 'dest'" class="popover popover-dest" role="dialog" aria-label="목적지 선택">
          <div class="popover-header">추천 목적지</div>
          <div class="dest-list">
            <button v-for="s in suggestions" :key="s.id" class="dest-row" @click.stop="selectDestination(s.name)">
              <span class="dest-icon" aria-hidden="true">🏙️</span>
              <span class="dest-texts">
                <strong>{{ s.name }}</strong>
                <small>{{ s.sub }}</small>
              </span>
            </button>
          </div>
        </div>
      </div>

      <!-- 보관기관(날짜) -->
      <div class="search-item" :class="{ active: openPanel === 'dates' }" @click="toggle('dates')">
        <label class="label">보관 기간</label>
        <input type="text" :value="dateLabel" placeholder="날짜 선택" readonly />

        <div
          v-if="openPanel === 'dates'"
          class="popover popover-dates"
          role="dialog"
          aria-label="날짜 선택"
          @click.stop
          @keydown.esc.prevent.stop="close()">
          <!-- 보관기간 달력 -->
          <div class="cal-head">
            <button class="nav-btn" @click.stop="prevMonth" aria-label="이전 달">‹</button>
            <div class="cal-title">{{ monthTitle(viewYear, viewMonth) }}</div>
            <button class="nav-btn" @click.stop="nextMonthNav" aria-label="다음 달">›</button>
          </div>

          <!-- 보관기관 달력 -->
          <div class="cal-1col">
            <Calendar
              :year="viewYear"
              :month="viewMonth"
              :start="startDate"
              :end="endDate"
              :hover="hoverDate"
              @pick="onPickDate"
              @hover="hoverDate = $event" />
          </div>

          <div class="date-actions">
            <button class="link-btn" @click.stop="clearDates">지우기</button>
            <div class="grow"></div>
            <button class="cta" :disabled="!startDate || !endDate" @click.stop="applyDates">적용</button>
          </div>
        </div>
      </div>

      <!-- 3) 짐 크기/개수 -->
      <div class="search-item size-item" :class="{ active: openPanel === 'bags' }" @click="toggle('bags')">
        <label class="label">보관함 크기/개수</label>
        <input type="text" :value="bagsLabel" placeholder="어떤 짐을 보관하시나요?" readonly />

        <div
          v-if="openPanel === 'bags'"
          class="popover popover-guests"
          role="dialog"
          aria-label="짐 크기/개수 선택"
          @keydown.esc.prevent.stop="close()">
          <div v-for="row in bagRows" :key="row.key" class="guest-row">
            <div class="guest-txt">
              <strong>{{ row.title }}</strong>
              <small>{{ row.desc }}</small>
            </div>
            <div class="guest-ctrl">
              <button class="circle" :disabled="counters[row.key] === 0" @click.stop="dec(row.key)">−</button>
              <span class="count">{{ counters[row.key] }}</span>
              <button class="circle" @click.stop="inc(row.key)">＋</button>
            </div>
          </div>

          <div class="date-actions">
            <button class="link-btn" @click.stop="resetBags">지우기</button>
            <div class="grow"></div>
            <button class="cta" @click.stop="applyBags">적용</button>
          </div>
        </div>
      </div>

      <!-- 검색 버튼 -->
      <button class="search-btn" aria-label="검색" @click="submit">
        <svg viewBox="0 0 24 24" width="22" height="22" aria-hidden="true">
          <path
            d="M21 21l-4.4-4.4M10.5 18a7.5 7.5 0 1 1 0-15 7.5 7.5 0 0 1 0 15Z"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round" />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, h, defineComponent } from "vue";
import { useRouter } from "vue-router"; // ✅ 추가

const router = useRouter(); // ✅ 라우터 사용

/* 패널 열기/닫기 */
const openPanel = ref(null);
const dockRef = ref();
function toggle(which) {
  openPanel.value = openPanel.value === which ? null : which;
}
function close() {
  openPanel.value = null;
}

/* 목적지 */
const destination = ref("");
const suggestions = [
  { id: 1, name: "부산", sub: "푸른 바다와 활기찬 해변으로 유명한 곳" },
  { id: 2, name: "광안리해수욕장", sub: "해변의 매력을 느낄 수 있는 곳" },
  { id: 3, name: "강릉시", sub: "자연을 만끽하기 좋은 곳" },
  { id: 4, name: "속초", sub: "호수로 인기 있는 곳" },
  { id: 5, name: "오사카시, 일본", sub: "관광 명소: 오사카성" },
  { id: 6, name: "전주", sub: "한옥마을과 다이닝을 즐기기 좋은 곳" },
  { id: 7, name: "제주도", sub: "폭포와 올레길로 유명한 관광 명소" },
];
function selectDestination(name) {
  destination.value = name;
  close();
}

/* 날짜 */
const dateTab = ref("range");
const startDate = ref(null);
const endDate = ref(null);
const hoverDate = ref(null);

const today = new Date();
const viewYear = ref(today.getFullYear());
const viewMonth = ref(today.getMonth());
const nextYear = computed(() => (viewMonth.value === 11 ? viewYear.value + 1 : viewYear.value));
const nextMonth = computed(() => (viewMonth.value === 11 ? 0 : viewMonth.value + 1));

function monthTitle(y, m) {
  return `${y}년 ${m + 1}월`;
}
function prevMonth() {
  viewMonth.value === 0 ? ((viewMonth.value = 11), viewYear.value--) : viewMonth.value--;
}
function nextMonthNav() {
  viewMonth.value === 11 ? ((viewMonth.value = 0), viewYear.value++) : viewMonth.value++;
}

function onPickDate(d) {
  if (!startDate.value || (startDate.value && endDate.value)) {
    startDate.value = d;
    endDate.value = null;
  } else if (d < startDate.value) {
    endDate.value = startDate.value;
    startDate.value = d;
  } else {
    endDate.value = d;
  }
}
function clearDates() {
  startDate.value = null;
  endDate.value = null;
  hoverDate.value = null;
}
function applyDates() {
  close();
}
function fmt(d) {
  return `${d.getMonth() + 1}월 ${d.getDate()}일`;
}
const dateLabel = computed(() =>
  startDate.value && endDate.value
    ? `${fmt(startDate.value)} - ${fmt(endDate.value)}`
    : startDate.value
    ? `${fmt(startDate.value)} - ...`
    : ""
);

/* 짐 크기/개수 */
const counters = reactive({ xs: 0, s: 0, m: 0, l: 0, xl: 0 });
const bagRows = [
  { key: "xs", title: "XS", desc: "에코백,백팩,서류가방" },
  { key: "s", title: "S", desc: "의류박스, 서랍형수납" },
  { key: "m", title: "M", desc: "여행가방, 의류박스" },
  { key: "l", title: "L", desc: "행거의류, 중형가전" },
  { key: "xl", title: "XL", desc: "대형짐, 스포츠장비" },
];
const inc = (k) => counters[k]++;
const dec = (k) => counters[k] && counters[k]--;
const resetBags = () => {
  Object.keys(counters).forEach((k) => (counters[k] = 0));
};
const applyBags = () => close();
const bagsLabel = computed(() => {
  const p = [];
  if (counters.xs) p.push(`XS ${counters.xs}`);
  if (counters.s) p.push(`S ${counters.s}`);
  if (counters.m) p.push(`M ${counters.m}`);
  if (counters.l) p.push(`L ${counters.l}`);
  if (counters.xl) p.push(`XL ${counters.xl}`);
  return p.join(" · ");
});

/* ✅ 제출 → /login 으로 이동 */
function submit() {
  router.push("/login");
}

/* 로컬 Calendar */
const Calendar = defineComponent({
  name: "Calendar",
  props: {
    year: Number,
    month: Number,
    start: Date,
    end: Date,
    hover: Date,
  },
  emits: ["pick", "hover"],
  computed: {
    grid() {
      const first = new Date(this.year, this.month, 1);
      const startIdx = (first.getDay() + 6) % 7;
      const daysInMonth = new Date(this.year, this.month + 1, 0).getDate();
      const cells = [];
      for (let i = 0; i < startIdx; i++) cells.push(null);
      for (let d = 1; d <= daysInMonth; d++) cells.push(new Date(this.year, this.month, d));
      while (cells.length % 7 !== 0) cells.push(null);
      return cells;
    },
  },
  methods: {
    isSame(a, b) {
      return a && b && a.toDateString() === b.toDateString();
    },
    inRange(d) {
      return this.start && this.end && d >= this.start && d <= this.end;
    },
    hovering(d) {
      if (!this.start || this.end || !this.hover) return false;
      const [min, max] = this.hover > this.start ? [this.start, this.hover] : [this.hover, this.start];
      return d >= min && d <= max;
    },
  },
  render() {
    const header = ["월", "화", "수", "목", "금", "토", "일"].map((d) => h("div", { class: "dow" }, d));

    const cells = this.grid.map((cell, i) =>
      h(
        "button",
        {
          key: i,
          class: {
            day: true,
            empty: !cell,
            start: cell && this.isSame(cell, this.start),
            end: cell && this.isSame(cell, this.end),
            inrange: cell && this.inRange(cell),
            hovering: cell && this.hovering(cell),
          },
          disabled: !cell,
          onMouseenter: () => this.$emit("hover", cell),
          onFocus: () => this.$emit("hover", cell),
          onClick: (e) => {
            e.stopPropagation();
            this.$emit("pick", cell);
          },
        },
        cell ? [h("span", null, cell.getDate())] : []
      )
    );

    return h("div", { class: "cal" }, [h("div", { class: "cal-grid" }, [...header, ...cells])]);
  },
});
</script>

<style lang="scss" scoped>
.search-dock > .search-bar {
  pointer-events: auto;
}

/* 바디 */
.search-bar {
  background: #fff;
  border: 1.5px solid #e5e7eb;
  border-radius: 9999px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 8px 14px;
  display: grid;
  align-items: center;
  grid-template-columns: 1.05fr 0.9fr 0.95fr auto;
  position: relative;
  // top: 100px;
  overflow: visible;
  z-index: 10;
}
.search-item {
  padding: 14px 22px;
  position: relative;
  display: grid;
  grid-auto-rows: min-content;
  align-content: center;
  min-height: 50px;
  cursor: pointer;
  z-index: 11;
}
.search-item input {
  color: #3e9c9b;
  font-weight: 400;
  font-size: 15px;
}
.search-item:not(:first-child)::before {
  content: "";
  position: absolute;
  left: 0;
  top: 12px;
  bottom: 12px;
  width: 1px;
  background: #e5e7eb;
}
.search-item.active {
  background: transparent;
  z-index: 20;
  transform: translateY(1px);
}

.label {
  font-size: clamp(16px, 1.5vw, 17px);
  font-weight: 700;
  color: #555353;
  margin-top: 7px;
  margin-bottom: 4px;
  line-height: 1;
  text-align: left;
}
input {
  border: none;
  outline: none;
  font-size: clamp(14.5px, 1.5vw, 15.5px);
  color: #b8b3b3;
  padding: 0;
  width: 100%;
  background: transparent;
  text-align: left;
}
input::placeholder {
  color: #a3a3a3;
}

.search-btn {
  width: 50px;
  height: 50px;
  border: none;
  border-radius: 50%;
  background: #3e9c9b;
  color: #fff;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  margin-left: 10px;
  box-shadow: 0 6px 16px rgba(2, 133, 135, 0.25);
  transition: background 0.2s ease, transform 0.05s ease;
}
.search-btn:hover {
  background: #3a8c88;
}
.search-btn:active {
  transform: translateY(1px);
}

/* 목,날,짐 클릭했을때 뜨는 box */
.popover {
  position: absolute;
  left: 22px;
  right: 22px;
  top: calc(100% + 12px);
  background: #fff;
  border-radius: 20px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.15);
  border: 1px solid #eee;
  padding: 14px;
  z-index: 30;
}

/* 목적지 */
.popover-dest {
  max-height: 300px;
  overflow: auto;
}
.popover-header {
  font-weight: 700;
  padding: 8px 10px 14px;
  color: #3e9c9b;
}
.dest-list {
  display: flex;
  flex-direction: column;
  gap: 5px;
}
.dest-row {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 10px;
  border-radius: 12px;
  width: 100%;
  border: 1px solid transparent;
  background: #fff;
  cursor: pointer;
}
.dest-row b,
.dest-row strong {
  display: inline-block;
  margin-right: 5px;
}
.dest-row small {
  color: #a3a3a3;
}
.dest-row:hover {
  background: #d8f1ea;
  border-color: #eee;
}
.dest-icon {
  width: 36px;
  height: 36px;
  flex: 0 0 36px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #eef2f7;
}

/* 날짜 */
:deep(.popover-dates) {
  padding: 18px 18px 15px;
}
:deep(.date-tabs) {
  display: flex;
  justify-content: center;
  gap: 8px;
  padding: 0 4px 0;
}
:deep(.chip) {
  border: none;
  background: #f2f3f5;
  border-radius: 999px;
  padding: 8px 14px;
  font-weight: 600;
  cursor: pointer;
}
:deep(.chip:hover) {
  background: #d8f1ea;
}
:deep(.chip.on) {
  background: #f2f3f5;
  border: 1px solid #eee;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
}

:deep(.cal-head) {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 15px;
  padding: 20px 6px;
  flex-wrap: nowrap;
  line-height: 1;
}
:deep(.cal-title) {
  text-align: center;
  font-weight: 700;
  font-size: 15px;
  white-space: nowrap;
}
:deep(.nav-btn) {
  width: 35px;
  height: 35px;
  flex-shrink: 0;
  border: none;
  border-radius: 999px;
  background: #f2f3f5;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s ease, transform 0.05s ease, box-shadow 0.15s ease;
}
:deep(.nav-btn:hover) {
  background: #d8f1ea;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.06);
}
:deep(.nav-btn:active) {
  transform: translateY(1px);
}

:deep(.cal-1col) {
  display: block;
  padding-top: 6px;
}
:deep(.cal) {
  background: #fff;
  border-radius: 16px;
}
:deep(.cal-grid) {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 4px;
  padding: 8px;
}
:deep(.dow) {
  text-align: center;
  font-weight: 700;
  color: #b8b3b3;
  padding: 6px 0;
}
:deep(.day) {
  height: 40px;
  border: none;
  border-radius: 10px;
  background: #fff;
  cursor: pointer;
  position: relative;
}
:deep(.day.empty) {
  background: transparent;
  cursor: default;
}
:deep(.day:hover:not(.empty)) {
  background: #f6f7f9;
}
:deep(.day.start),
:deep(.day.end) {
  background: #3e9c9b;
  color: #000;
} // 시작, 끝
:deep(.day.inrange) {
  background: #3e9c9b;
} // 구간(시작~끝 사이)
:deep(.day.hovering) {
  background: #d8f1ea;
} // 마우스 드래그 중일 떄

/* 액션 */
.date-actions {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 8px 2px;
}
.link-btn {
  background: transparent;
  border: none;
  color: #b8b3b3;
  text-decoration: underline;
  cursor: pointer;
}
.grow {
  flex: 1;
}
.cta {
  padding: 10px 16px;
  border: none;
  border-radius: 12px;
  background-color: #3e9c9b;
  color: #fff;
  font-weight: 500;
  cursor: pointer;
}

/* 짐 */
.popover-guests {
  padding: 12px;
  min-width: 420px;
  display: flex;
  flex-direction: column;
}
.guest-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 14px 8px;
  border-bottom: 1px solid #f0f0f0;
}
.guest-row:last-child {
  border-bottom: none;
}
.guest-txt {
  display: flex;
  align-items: center;
}
.guest-txt strong {
  margin-left: 15px;
  color: #3e9c9b;
  font-size: 18px;
  font-weight: 700;
  display: inline-block;
  min-width: 35px;
}
.guest-txt small {
  color: #787474;
  font-size: 14px;
  white-space: nowrap;
}
.guest-ctrl {
  display: flex;
  align-items: center;
  gap: 12px;
}
.count {
  width: 22px;
  text-align: center;
}
.guest-ctrl .circle {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: 1px solid #e1e1e5;
  background: #fff;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.15s ease, border-color 0.15s ease, color 0.15s ease, transform 0.05s ease;
}
.guest-ctrl .circle:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
.guest-ctrl .circle:first-child,
.guest-ctrl .circle:last-child {
  color: #000;
}
.guest-ctrl .circle:first-child:hover,
.guest-ctrl .circle:last-child:hover {
  background: #d8f1ea;
}

@media (max-width: 1480px) {
  .search-dock {
    width: calc(100% - 40px);
  }
  .search-bar {
    padding: 10px 12px;
    grid-template-columns: 1fr 0.9fr 0.9fr auto;
  }
  .search-item {
    padding: 12px 16px;
  }
  .popover {
    left: 16px;
    right: 16px;
  }
}

@media (max-width: 1440px) {
  .search-dock {
    width: calc(100% - 40px);
  }
  .search-bar {
    padding: 8px 12px;
    grid-template-columns: 1fr 0.9fr 0.9fr auto;
  }
  .search-item {
    padding: 12px 16px;
  }

  .popover {
    left: 16px;
    right: 16px;
  }
}

@media (max-width: 1420px) {
  .search-dock {
    width: calc(100% - 40px);
  }
  .search-bar {
    padding: 8px 12px;
    grid-template-columns: 1fr 0.9fr 0.9fr auto;
  }
  .search-item {
    padding: 12px 16px;
  }

  .popover {
    left: 16px;
    right: 16px;
  }
}
@media (max-width: 1350px) {
  .search-dock {
    width: calc(100% - 80px);
  }
  .search-bar {
    padding: 10px 12px;
    grid-template-columns: 0.7fr 0.6fr 0.7fr auto; // 서치바 안에 내용물
  }
  .search-item {
    padding: 12px 16px;
  }
  .popover,
  .popover-dates,
  .popover-guests {
    max-width: 480px;
  }
}
@media (max-width: 1280px) {
  .search-dock {
    // top: 390px;
    // left: 50%;
    // transform: translate(-50%);
    width: calc(100% - 80px);
  }
  .search-bar {
    padding: 10px 12px;
    grid-template-columns: 0.7fr 0.6fr 0.7fr auto; // 서치바 안에 내용물
  }
  .search-item {
    padding: 12px 16px;
  }

  .popover,
  .popover-dates,
  .popover-guests {
    max-width: 480px;
  }
}
@media (max-width: 1220px) {
  .search-dock {
    // top: 375px;
    // left: 50%;
    // transform: translate(-50%);
    width: calc(100% - 80px);
  }
  .search-bar {
    padding: 10px 12px;
    grid-template-columns: 0.7fr 0.6fr 0.7fr auto;
  }
  .search-item {
    padding: 12px 16px;
  }

  .popover,
  .popover-dates,
  .popover-guests {
    max-width: 380px;
  }
}
@media (max-width: 1180px) {
  .search-dock {
    // top: 360px;
    // left: 50%;
    // transform: translate(-50%);
    width: calc(100% - 80px);
  }
  .search-bar {
    padding: 10px 12px;
    grid-template-columns: 0.7fr 0.6fr 0.7fr auto;
  }
  .search-item {
    padding: 12px 16px;
  }
}
@media (max-width: 1150px) {
  .search-bar {
    padding: 10px 12px;
    grid-template-columns: minmax(0, 0.95fr) minmax(0, 0.75fr) minmax(0, 0.85fr) auto;
    column-gap: 6px;
  }
  .search-item {
    padding: 12px 14px;
  }
  .search-item input {
    min-width: 0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  // search 바 사이 before
  .search-item:not(:first-child)::before {
    height: 50px;
    top: 10px;
    bottom: 8px;
  }
  .search-item.size-item .popover,
  .search-item.storage-item .popover {
    right: -70px;
    width: 380px;
  }
  .search-item .popover {
    left: 0;
    right: 0;
    width: auto;
    max-width: none;
    border-radius: 18px;
  }
  .search-item .popover-guests {
    min-width: 0;
  }
  .guest-row {
    gap: 10px;
  }
  .guest-txt strong {
    font-size: 16px;
    // min-width: 30px;
    margin-left: 10px;
  }
  .guest-txt small {
    white-space: nowrap;
  }
}

@media (max-width: 1060px) {
  .search-bar {
    grid-template-columns: minmax(0, 0.7fr) minmax(0, 0.7fr) minmax(0, 0.6fr) auto;
    column-gap: 4px;
  }
  .search-item {
    padding: 11px 12px;
  }
  .search-item:not(:first-child)::before {
    height: 46px;
    top: 8px;
  }
  .search-item .popover {
    left: 0;
    right: 0;
    width: auto;
    max-width: none;
  }
  .search-item.size-item .popover,
  .search-item.storage-item .popover {
    right: -70px;
    width: 330px;
  }
  .dest-row {
    padding: 5px 10px;
  }
  .dest-row small {
    font-size: 10px;
  }
  .guest-row {
    gap: 8px;
  }
  .guest-txt strong {
    font-size: 15px;
    margin-left: 6px;
  }
}
@media (max-width: 1055px) {
  .dest-row small {
    display: none;
  }
}

@media (max-width: 980px) {
  .search-item .popover {
    left: 0;
    right: 0;
    width: auto;
    max-width: none;
  }
  .search-item.size-item .popover,
  .search-item.storage-item .popover {
    right: -70px;
    width: 300px;
  }

  .guest-ctrl {
    gap: 5px;
  }
}

@media (max-width: 850px) {
  .search-item:nth-child(2),
  .search-item:nth-child(3) {
    display: none;
  }

  .search-bar {
    grid-template-columns: 1fr auto;
    width: 90%;
    margin: 0 auto;
  }

  .search-item {
    padding: 10px 14px;
  }
  .search-item:first-child .popover {
    width: 95%;
    margin: 0 auto;
    left: 10px;
  }
  .popover-header {
    font-size: 18px;
  }
  .dest-texts {
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .dest-texts strong {
    font-size: 15px;
  }
  .dest-texts small {
    display: inline-block;
    color: #a3a3a3;
    font-size: 12px;
    white-space: nowrap;
  }
}

@media (max-width: 768px) {
  .search-item:first-child .popover {
    width: 95%;
    margin: 0 auto;
    left: 20px;
  }
}

@media (max-width: 390px) {
  .search-bar {
    grid-template-columns: 1fr auto;
    width: 90%;
    margin: 0 auto;
  }
  .search-item {
    padding: 4px 20px;
  }
  .label {
    font-size: 15px;
  }
  .search-bar input {
    font-size: 13px;
    margin-bottom: 3px;
  }
  .popover-header {
    font-size: 16px;
    padding: 8px 10px 14px;
  }
  .dest-list {
    gap: 0;
  }
  .dest-texts strong {
    white-space: nowrap;
    color: #000;
  }
  .dest-texts small {
    display: none;
  }

  .search-item:first-child .popover {
    width: 120%;
    margin: 0 auto;
    left: 5px;
  }
  .popover-dest {
    padding: 8px;
  }
}

// @media (max-width: 767px) {
//   .search-dock {
//     display: none;
//   }
// }
</style>
