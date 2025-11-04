<template>
  <section class="service-section">
    <div class="inner">
      <!-- 왼쪽 텍스트 -->
      <div class="left">
        <h2>마타주 제공 서비스</h2>
        <p>마타주에서 다양한 서비스를 이용해 보세요</p>
      </div>

      <!-- 오른쪽: 카드 + 자세히 보러 가기 -->
      <div class="right">
        <div class="cards">
          <div class="card" v-for="(item, i) in services" :key="i" :class="item.class">
            <h3 v-html="item.title"></h3>
            <p class="desc">{{ item.desc }}</p>
            <div class="img-wrap">
              <img :src="item.img" :alt="item.title" loading="lazy" />
            </div>
          </div>
        </div>

        <!-- ✅ '자세히 보러 가기' — 요금 섹션과 완전히 동일 규칙 -->
          <!-- <div class="more-row">
            <router-link class="more" :to="moreLink">자세히 보러 가기 →</router-link>
          </div> -->
      </div>
    </div>
  </section>
</template>

<script setup>
const moreLink = "/information";

const services = [
  { title: "보관", desc: "짐이 많을 땐 잠시 맡기세요", img: "/images/mains/about/image 466.png", class: "storage" },
  { title: '배송<span class="accent">(집 → 지점)</span>', desc: "무거운 짐은 문 앞에서 해결하세요", img: "/images/mains/about/Group 405.png", class: "house-to-branch" },
  { title: '배송<span class="accent">(지점 → 집)</span>', desc: "마지막까지 편하게 여행하세요", img: "/images/mains/about/Group 406.png", class: "branch-to-house" },
];
</script>

<style scoped lang="scss">
:root { --accent-color:#de3464; }

/* ===== 기존 스타일 유지 ===== */
.service-section {
  width: 100%;
  padding: 120px 0 80px;
  display: flex;
  justify-content: center;
  font-family: "Pretendard","Inter",sans-serif;

  @media (max-width:1080px){ text-align:center; }
  @media (max-width:390px){ padding-top:100px; }
  @media (max-width:768px){ padding: 80px 0 80px; }

  .inner{
    display:flex;
    align-items:flex-start;
    justify-content:center;
    gap:80px;
    max-width:1320px;
    width:100%;
    padding:0 40px;

    @media (max-width:1386px){ display:block; }
  }

  .left{
    flex:0 0 340px;
    h2{ font-size:clamp(24px,2.5vw,35px); font-weight:700; color:#111; line-height:1.3; }
    p{ font-size:clamp(15px,1.4vw,18px); color:#999; line-height:1.6; }
  }

  .right{ width:100%; }

  .cards{
    display:grid;
    grid-template-columns:repeat(3,1fr);
    gap:34px;

    @media (max-width:1358px){ padding-top:5.0001%; }
    @media (max-width:1080px){ grid-template-columns:repeat(1,1fr); gap:28px; }
    @media (max-width:1080px){ display:block; max-width:290px; margin:auto; }
    @media (max-width:390px){ padding-top:50px; }
  }

  .card{
    max-width:290px; background:#fff; border:1px solid #e8e8e8; 
    border-radius:20px;
    box-shadow:0 2px 6px rgba(0,0,0,.05); display:flex; flex-direction:column; align-items:center;
    text-align:center; padding:36px 16px 22px; transition:all .3s ease;

    @media (max-width:978px){ margin-top:10px; margin-bottom: 20px;}

    &:hover{ transform:translateY(-3px); box-shadow:0 6px 12px rgba(0,0,0,.08); }

    h3{ font-size:clamp(25px,2.6vw,27px); font-weight:700; color:#111; margin-bottom:4px; }
    .desc{ font-size:clamp(14px,1.3vw,15px); color:#333; margin:0 0 40px; line-height:1.5; }

    .img-wrap{
      display:flex; align-items:center; justify-content:center; margin:0 0 14px; line-height:0;

      img{ width:auto; height:auto; object-fit:contain; display:block; }
    }
  }

  .storage img{ max-height:150px; margin-top:-14px !important; margin-bottom:17px !important; }
  .house-to-branch img{ max-height:125px; margin-top:-14px !important; margin-bottom:18px !important; }
  .branch-to-house img{ max-height:135px; transform:scale(1.01); margin-top:8px !important; margin-bottom:20px !important; }

  @media (max-width:978px){ padding-bottom:0; }
  @media (max-width:768px){
    .card{ width:100%; max-width:320px; }
  }
}

/* 강조 색상 */
:deep(.accent){ color:#de3464 !important; font-size:.8em !important; font-weight:700; margin-left:5px; }

/* ====== '자세히 보러 가기' — 요금 섹션과 동일(항상 우측 정렬) ====== */
.more-row{
  display:flex;
  justify-content:flex-end;   /* ✅ 모든 뷰포트에서 우측 정렬 고정 */
  margin-top:12px;
}
.more{
  color:#777;
  font-size:clamp(17px,1.3vw,17px);  /* ✅ 요금 섹션과 동일 */
  text-decoration:none;
  margin:0;   
  margin-top: 20px;                       /* ✅ 여백도 동일 처리 */
}
.more:hover{ text-decoration:underline; }
</style>
