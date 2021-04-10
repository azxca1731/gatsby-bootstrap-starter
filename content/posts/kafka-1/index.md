---
title: 카프카 스터디 3장 리뷰
date: '2020-08-14T20:20:49.891Z'
layout: post
path: '/kafka-1/'
category: kafka
tags:
  - Kafka
  - Study
  - Review
---

<span class="text-danger">_이 리뷰는 카프카, 데이터 플랫폼의 최강자 책을 보고 리뷰하는 포스트입니다._</span>

## 카프카 디자인의 특징

1. 분산시스템
2. 페이지 캐시
3. 배치 전송 처리

### 분산 시스템

카프카는 분산시스템입니다. 그렇기에 유동적으로 서버 댓수를 늘릴 수가 있습니다. 만약 한대가 초당 3000개의 메시지를 처리 할 수 있고 처리 할량이 초당 12000개라면 4대로 구성을 하여서 운행을 하면됩니다.

### 페이지 캐시

카프카는 페이지 캐시를 이용합니다. 카프카는 OS가 페이지 캐시를 이용하도록 디자인 되어있습니다. 그렇기에 SATA 디스크를 운영해도 괜찮습니다

_카프카는 페이지 캐시를 이용하기에 메가비트 단위를 처리하는 시스템이여도 5기가까지 힙으로 구성하는 것을 권장하며 남은 메모리는 페이지 캐시로 운영을 하는 것이 좋습니다. 또한 페이지 캐시를 OS에서 받아서 하는 것이기에 한 시스템에 카프카 하나만 뛰어놓는 것을 권장합니다._

### 배치 전송 처리

모든 데이터 처리에 있어서 병목은 IO일 것입니다. 그렇기에 카프카는 작은 IO등을 묶어서 처리 할 수 있도록 배치 작업으로 처리합니다. 그렇기에 카프카는 빠를 수 있습니다.
_하지만 그렇기에 producer에서 보내는 즉시 consumer로 간다는 것을 보장 할 수 없습니다._

<!--more-->

---

## 카프카 데이터 모델

### 토픽

카프카는 토픽이라는 것으로 분류해서 데이터를 구분하고 처리합니다. 토픽은 다음과 같은 특징을 가집니다.

1. 토픽의 이름은 249자 미만 영문, 숫자, '.', '\_', '-' 로 구성됩니다.
2. 하나의 서비스에 독립적으로 쓰이는 것이 아니라면 각 서비스마다 토픽의 이름에 접두어를 두어 중복을 피하는 것이 좋습니다.

### 파티션

파티셔닝을 하여 동시에 많은 데이터를 처리 할 수 있습니다. 파티션이 느는 만큼 동시에 많은 데이터를 카프카에 프로듀스 할 수 있습니다.
그렇다면 파티션을 일단 많이 만드는 것이 좋을까요?

#### 파티션을 늘림으로서 얻는 단점

1. 파일 핸들러의 낭비
   > 파티션은 브로커의 디렉토리와 매핑이 되고, 실제 데이터,인덱스와 연결이 되어 파티션당 2개의 파일 핸들러가 있게 됩니다.
2. 장애 복구 시간 증가
   > 파티션마다 리더가 있기에 컨트롤러에 의해 리더가 선출되는 것이 5ms이라 해도 1000개의 파티션에서 리더를 선출한다면 5초가 걸리게 됩니다. 또한 컨트롤러 브로커가 죽게 된다면 모든 파티션의 데이터를 읽어야하는데 10000개의 파티션이 있고 개당 2ms가 걸린다 해도 20초라는 시간이 걸리게 됩니다.

#### 적절한 파티션 수는?

초당 얼마나 처리할지 결정하여 정하는 것이 좋습니다. 또한 카프카에서는 브로커당 2000개 정도의 파티션 수를 권장합니다.
또한 파티션은 늘리는 것은 가능하지만 줄이는 것은 가능하지 않습니다. 그렇기에 적절하게 설정하고 차근차근 늘려가는 것을 권장합니다.

### 오프셋과 메시지 순서

파디션마다 저장되는 위치를 오프셋이라고 부릅니다. 오프셋은 파티션내에 유일하고 64비트 정수형태입니다. 파티션마다 오프셋은 유니크하며 파티션내의 순서는 보장됩니다.

_그렇기에 파티션이 하나인 토픽에 대해서는 순서를 보장합니다. 그리고 그 파티션의 갯수가 한개를 초과한다면 순서를 보장할 수 없습니다._

---

## 카프카의 고가용성

### 리플리케이션 팩터와 리더, 팔로워

리플리케이션 팩터는 디폴트가 있으며 토픽마다 설정이 가능합니다. 이 리플리케이션을 이용해 높은 가용성을 얻을 수 있습니다.
만약 2개의 브로커가 있고 하나의 토픽이 있고 하나의 파티션이 있다고 합시다. 이 때 리플리케이션 팩터가 2이면 복제가 되 관리가 됩니다. 그렇기에 유지가 가능합니다.
그러면 리더, 팔로워는 뭘까요?

1. 리더
   > 읽고 쓰기를 관리함.
2. 팔로워
   > 주기적으로 리더에게서 컨슈머처럼 가져옴. 그리고 그 정보를 복사해서 가지고 있음.

리플리케이션 팩터만큼 추가 여유분을 가지고 있음으로 그만큼 더 많이 저장공간을 소모한다

### 리더와 팔로워의 관리

리더와 팔로워의 관리는 ISR(In Sync Replica)라는 개념으로 관리합니다.

1. 팔로워가 죽는 경우
   > 리더가 정해진 시간(replica.lag.time.max.ms)만큼 확인 요청이 안온다면 ISR에서 제외시킵니다.
2. 리더가 죽는 경우
   > ISR에서 남은 팔로워가 리더를 맡게 됩니다.
3. 모두 죽는 경우
   > 모두 죽는 경우는 두가지로 나뉩니다.
   > 만약 모두 죽는 경우, 마지막 리더가 살아나기를 기다리거나, ISR에서 추방되었지만 먼저 살아나는 경우가 있습니다.
   > 마지막 리더가 살아나기를 기다린다면 데이터는 모두 유지가 됩니다. 하지만 다른 브로커가 살아나도 카프카는 작동하지 않습니다.
   > ISR에서 추방되었지만 먼저 살아나는 경우는 그 브로커가 리더가 되고 그 리더가 가지고 있는 정보로 ISR의 데이터는 구성이 됩니다.
   > 그러므로 없어지는 데이터가 있을 수 있습니다.

카프카 0.11.0.0 이하 버전은 빠른 회복이 기본이고 그 상위 버전은 마지막 리더가 디폴트입니다. 그러기에 _unclean.leader.election.enable_ 옵션을 설정하여 확인 할 수 있습니다.(true면 2번 false면 1번)

---

## 카프카에서 사용하는 주키퍼 지노드 역할

카프카의 중요한 지노드가 몇가지가 있습니다. 살펴보겠습니다.

1. 컨트롤러
   > 리더 변경을 책임지고 브로커중에 하나가 컨트롤러의 역할을 맡습니다.
2. 브로커
   > 클러스터 내의 토픽정보, 토픽의 파티션 수, ISR 구성 정보, 리더 정보등을 확인 가능합니다.
3. 컨슈머
   > 파티션 오프셋 정보가 저장이 됩니다.
4. 컨피크
   > 카프카의 설정이 저장됩니다.