# project_start

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```
# vue_team2


==============변경해야할 것(완료시 체크하기)========================
10-14 index의 seo 변경 & public의 파비콘 이미지파일 바꾸기
10-17 views>mains 안의 파일들을 components>main-com으로 옮기기. 옮긴 후 라우터 import 바꾸기


===========상의해봐야 할것========================
만들 서비스들을 구체화 > 질문 대비.

================================

npm install @vuepic/vue-datepicker

ㄴ----이거버전은 @vuepic/vue-datepicker@11.0.3



===============
카드 스타일 패딩이나 보더라디우스
=======================
-------------------------------------
혜주님 후기페이지
-------------------------------------
큰 카드   
  border: 1px solid #eee;
    *border-radius: 12px;*    
    padding: 16px 18px;
    background: #fff;
작은카드
 *border-radius: 8px;*    
 
 더보기 버튼
 .loadmore[data-v-7ac975f5] {
    min-width: 160px;
    height: 40px;
    padding: 0 16px;
    *border-radius: 10px;*    
    border: 1px solid #E5E7EB;
    background: #fff;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
}

-----------------------------------------
정은님 이용안내의 카드
-------------------------------------
    .info-steps .step-card[data-v-f1032799] {
        max-width: none;
        padding: 18px 16px;
        *border-radius: 14px;*    
        box-sizing: border-box;
    }

    사물함 사이즈
.detail[data-v-cbbada12] {
    display: grid
;
    grid-template-columns: minmax(280px, 360px) 1fr;
    gap: 22px;
    padding: 18px;
    *border-radius: 16px;*   
    background: #fff;
    border: 1px solid #eef2f7;
    box-shadow: 0 2px 8px rgba(2, 6, 23, .04);
    align-items: center;
    padding-bottom: 30px;
}

보관불가품목
    .ban-list[data-v-46d92daa] {
        padding: 14px 16px;
        gap: 16px;
    }

.ban-list[data-v-46d92daa] {
    max-width: 1100px;
    display: flex
;
    align-items: center;
    background-color: #fff;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.08);
    gap: 30px;
    *border-radius: 20px;*      
    padding: 15px 18px;}
    
------------------------------
고객센터
-------------------------------------
맨위 카드 3장
.card[data-v-803e87a5] {
    width: 290px;
    height: 290px;
    background: #fff;
    border: 1px solid #e6eae9;
    *border-radius: 12px;*      
    display: flex
;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    transition: transform .2s ease, box-shadow .2s ease;
    padding: 10% 2%;
}

자주묻는질문 탭
.tab[data-v-803e87a5] {
    font-size: clamp(16px, 2vw, 18px);
    font-weight: 500;
    border: 1px solid #dcdedc;
    *border-radius: 6px;*      
    padding: 10px 28px;
    background: #fff;
    color: #6b7a76;
    transition: all 0.2s ease;
    cursor: pointer;
}

페이지네이션 버튼
.ghost[data-v-803e87a5] {
    font-size: 14px;
    border: 1px solid #e2e5e4;
    background: #fff;
    padding: 8px 20px;
    *border-radius: 4px;*      
    cursor: pointer;
    color: #000;
}
-------------------------
메인페이지
-------------------------------------
배공안내카드 어바웃??
.service-section .card[data-v-2751b56a] {
    max-width: 290px;
    background: #fff;
    border: 1px solid #e8e8e8;
    *border-radius: 14px;*          
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
    display: flex
;
    flex-direction: column;
    align-items: center;
    text-align: center;
    padding: 36px 16px 22px;
}

리뷰카드
review_card {
  width: 100%;
  height: 100%;
  background: #fff;
  *border-radius: 20px;*        
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.1);
  padding: 1.5rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  text-align: left;
}

이용요금 버튼
.info-main .tabs[data-v-1f1b3663]
특수성: (0,3,0)
 {
    display: inline-flex;
    background: #f9fafa;
   *border-radius: 6px;*    
    overflow: hidden;
    border: 1px solid #d9e1e1;
    margin: 0 auto clamp(28px, 4vw, 44px);}
마타주 지점찾기
제일큰박스
.search-main .searchbar[data-v-18fc7b07] {
    display: flex
;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    max-width: 560px;
    background: #fff;
    padding: 10px;
    *border-radius: calc(var(--radius) + 4px);*      

    border: 1px solid #f0f3f3;
    box-shadow: 0 4px 24px rgba(0, 0, 0, 0.06);
    gap: 10px;}

안의박스1
.search-main .searchbar input[data-v-18fc7b07], .search-main .searchbar select[data-v-18fc7b07] {
    flex: 1;
    height: 46px;
    border: 1px solid #e7efef;
    *border-radius: var(--radius);*      
    padding: 0 14px;
    font-size: 15px;
    outline: none;
    transition: border-color 0.15s ease, box-shadow 0.15s ease;}

    안의박스2
.search-main .cta[data-v-18fc7b07] {
    flex-shrink: 0;
    height: 46px;
    padding: 0 18px;
    border: none;
    *border-radius: var(--radius);*      
    background: var(--mint);
    color: #fff;
    font-weight: 700;
    font-size: 14px;
}

-----------------------
정리 가장큰것은 20px 디자인이들어간..메인리뷰스와이퍼 인포 물품안내에쓰임

맨위 서비스안내카드도 쓰면 어떨까함

프로모션도 비슷하게 좀더 라디우스를 주면 느낌이 많이 바뀔지고민..
그리고프로모션과 로그인페이지의 배너의 사각 라디우스를 맞췄으면 함

고객센터의 가장 큰 카드도 바꾸면 어떨까함




-------------------
예약하기 작은 1234 큰 123


큰보더라디우스 10px//12픽셀로 해야겟다 후기랑 맞추기
 작은보더라디우스 6px 으로 설정하기
 큰게 12면 8정도가 적당하지 않을까..

border-radius: 12px;

border-radius: 8px;

페이지..

---------------
알림창 수정해야함



------
가진 모달 종류..
AlertModal.vue 간단한 알림창
ConfirmModal.vue 완료창
AddressPicker.vue 카카오 주소검색
BranchSelectModal.vue 지점검색
FindID.vue 아이디찾기모달
FindPw.vue 비번찾기모달
ConfirmReserv.vue  레저베이션1 선택후 창으로 보여줌 -모달
FinishPayment.vue 레저베이션2 후 창으로 결제와 예약번호를 보여줌
ReserAgree.vue 레저1 작성후 창으로 약관 동의하기

--------
페이지 종류..

ChangeReserv.vue 예약변경 페이지
Reservation.vue  예약입력페이지
Reservation2.vue  예약 결제페이지
Reservation3.vue  예약완료페이지
Login.vue   로그인 페이지
Signup.vue    회원가입 페이지1 번호인증
Signup2.vue  회원가입 페이지2 입력및 완료

--------
컴포넌트 종류..
Reserv1Locker.vue  예약1의 사물함대여
Reserv2Arrival.vue  예약1의 짐 보내기
Reserv3Luggage.vue  예약1의 짐 배송받기
Reserv4Summary.vue 예약1의 예약 한눈에보기
Stepper.vue  예약123 과정을 보여줌


좀 헷갈리는것들...
Reser_check.vue  
라우터에는 있지만 쓰이지는 않음

FindReservModal.vue
src/views/sign/FindResarv.vue에서 사용됩니다. (모달로 열림)
어디서 오고(열리고) 어디로 가는지: FindResarv 페이지에서 "예약번호 찾기" 버튼으로 모달을 열고(같은 페이지), 모달은 예약번호(foundCode)를 보여주고 닫기(close)를 부모에 emit합니다. 모달 자체는 다른 페이지로 네비게이션을 수행하지 않습니다.

FindResarv.vue
— 예약번호 확인 페이지(뷰).
어디서 오는지: 라우터에 매핑된 경로(route)를 통해 진입합니다(정확한 경로명은 router 설정 확인 필요).
이 페이지에서 다음으로 넘어가는 곳:
"예약번호 찾기" 버튼 → FindReservModal.vue 모달 열기(v-if="showModal").
"예약 변경하기" 링크 → RouterLink로 "/changereserv" 경로로 이동.
폼 제출(checkReservation) 현재는 네비게이션 없음 — 전역 $alert만 호출(즉, 자동 이동 없음).

Reser_check.vue
 예약 확인(결제 전) 화면(뷰/라우트)로 사용됩니다.
이 페이지로 오는 곳: 예약 폼/예약 흐름의 이전 페이지(보통 Reserve 페이지)에서 이동합니다 — 정확한 출처는 프로젝트에서 checkoutPayload를 localStorage에 쓰는 파일을 찾아보면 확정됩니다.
이 페이지에서 가는 곳: finish() 버튼 클릭 시 window.location.href = "/complete"로 이동하여 Complete.vue(결제 완료)로 넘어갑니다.
이 컴포넌트는 mounted 시 localStorage의 "checkoutPayload"를 읽어 화면을 렌더링합니다(따라서 해당 키를 설정하는 페이지가 이전 화면입니다).

Complete.vue
이 페이지(Complete.vue)는 결제 완료 후 이동되는 페이지입니다.
주로 Reser_check.vue(예약 확인/결제 화면)에서 넘어옵니다.
이 페이지의 다음 이동은 "예약 확인" 버튼 클릭 시 "/reserve" 라우트로 이동하도록 되어 있습니다(홈으로 가기 버튼은 주석 처리됨).

SearchSpot.vue
아마미완인 서치스팟?
파일은 지점(브랜치) 선택용 모달 컴포넌트입니다.
제공된 파일들만으로는 이 컴포넌트가 정확히 어느 부모(페이지)에서 사용되는지 확정할 수 없습니다.
일반적인 흐름은: 예약(Reserve) 페이지에서 지점 선택을 위해 이 모달을 열고, 선택 후 예약 확인(Reser_check.vue) → 결제 완료(Complete.vue) 순서입니다.

ReserveForm.vue 이건 참고용 이전예약 *지우지않기*

│       ├─     └─      ─ 