---
title: 프론트 엔드 최적화
date: '2018-11-19T05:02:08.208Z'
layout: post
path: '/FrontendPerformanceOptimization/'
category: Frontend
tags:
  - Blockchain
  - Febble
---

웬 최적화는 로딩 최적화 + 렌더링 최적화로 나타낼 수 있고 각각의 최적화를 이루게 된다면 많은 것을 받을 수 있다.

##로딩 최적화
빠른 로딩속도는 사업 확장및 사용자 유치에 도움이 된다.

- 핀터레스트로 보는 성능 향상의 예시
> 44퍼센트의 매출, 40퍼센트의 유저, 50퍼센트의 클릭수의 증가를 유치하였다.

<!--more-->

###프론트 엔드 타이밍
돔 컨텐트 로디듣(Process, html,css,js파싱완료), 타 컨텐트 로드를 최대한 빠르게 하는 것을 목표로 한다.
css,js로 인한 html 파싱중단 - > 블록 리소스

브라우저는 어떻게 로드를 하는가
1. HTML로 돔트리 생성
2. CSSOM 트리 생성
3. 렌더 트리 생성
4. 레이아웃
5. 페인팅
6. 컴포즈
? 왜 자바 스크립트, css가 블럭리소스 인가 -> 이유 dom에 접근 권한이 있기 때문에
그래서 이 2개의 리소스의 최적화가 좋다.

###블럭 컨텐츠 최적화
1. 스크립트 파일 최적화
스크립트 파일은 바디 밑에다가 놔야지 렌더후에 로딩이 된다.
아니면 async로 블록을 하지 않게 해야한다.
그래야 흰화면을 안보여 줄 수 있다.
이래야 로딩바라도 빨리 보여 줄 수 있다.

2. CSS파일 최적화
실제로 다운로드 시간은 길지 않으나 TFFB(waiting)가 길다. 이것을 줄이기 위해서 외부 스타일 html에 인라인 스타일로 바꿔서 다운로드 시간을 줄인다.

---

###사용자 기준 최적화
구글 기준에서 몇가지 정의 해놓은 것이 있는데 주목해야할 3가지가 있다.
1. First Paint
>흰화면이 아닌 화면이 로딩
2. First Contentful Event
>사용자가 보기에 의미가 있는 컨텐츠 로딩 ! 이 시간을 최대한 줄이는 것을 목표로 삼는다
3. Time to Interactive

SSR은 2번을 최대한 줄이기위한 프레임워크의 방법이다.
이중 프리렌더러는 빌드타임에 생성이되고 ssr은 런타임에 생성이되는데 프리렌더러를 쓰는 경우가 더 좋다.

prerender-loader를 쓰면 fmp(first meaningful content)이 1초만에 나옰 수 있다.

---

###PWA
PRPL 패턴
PWA = Web + App
lazy loading

1. PWA 사례 1
> BookMyShow
> 앱사이즈 맟 월방문자 로딩타임을 줄여 80퍼센트의 전환율을 가졌다.
2. PWA 사례 2
> MakeMyTrip
> 월방문자 및 엄청난 이탈율 저하를 이룸


---

##렌더링 최적화
1. 레이아웃 쓰레싱
>javascript는 dom과 css를 변경시킴
>강제 동기 레이아웃은 위에 말한 것이 읽기만 해도 일어날 수 있고 이것이 빈번하게 일어나는 경우 늦어진다.
>이것이 레이아웃 쓰레싱이다.
>Dom에 접근하는경우 캐슁을 해놔서 쓰는 것이 현명하고 강제 동기 레이아웃을 막을수 있다.

2. Virual Dom (가상돔)
>돔 변경을 최소화

3. Web Worker (웹 워커)
> 60fps을 구현한다고 하자 그러면....<br /> 
> 60fps -> 16ms/fr(계산결과) -> 10ms/fr(브라우저계산을 생각한다면)

무거운 작업이 있으면 웹 워커를 사용해라!