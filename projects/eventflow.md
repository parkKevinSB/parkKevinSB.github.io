---
title: eventflow 실행 프레임워크
description: 장시간 수행되는 장비 작업을 Task·Process·Sequence 단위로 구성하고 이벤트 기반으로 진행하는 Java 실행 프레임워크입니다.
permalink: /projects/eventflow/
period: 2025
category: 작업 실행 프레임워크
role: 구조 설계 및 전체 개발, AYTSS 최초 적용
stack: Java 8, Event-driven Architecture, Multi-threading, SLF4J
---

## 개발 대상

`eventflow`는 장비 작업의 실행 순서와 상태 관리를 메시지 수신 코드에서 분리하기 위해 개발한 Java 프레임워크입니다. AYTSS와는 별도 모듈로 구성했으며, AYTSS의 차량 작업 실행 구조에 처음 적용했습니다.

| 실행 단위 | 역할 |
|---|---|
| EventHub | 이벤트 종류별 Listener 등록과 동기·비동기 전달 |
| EventHandler | 수신 이벤트 검증 및 업무 상태 반영 |
| Task | 독립 실행 Thread와 Process 목록 관리 |
| Process | 하나의 작업을 구성하는 PreCheck와 Sequence 순서 관리 |
| Sequence | 명령 실행, 이벤트 대기, 재시도와 완료 조건 처리 |
| TaskRegistry | Task 등록, 초기화, 시작과 중지 |

## 실행 구조

<ol class="flow-list">
  <li>Task가 설정된 주기로 실행 대상 Process를 확인합니다.</li>
  <li>Process는 실행 전 PreCheck와 Sequence Queue를 관리합니다.</li>
  <li>Sequence는 단계별 명령을 수행하고 필요한 Event Listener를 등록합니다.</li>
  <li>외부 시스템이나 장비 상태 이벤트를 수신하면 다음 단계 진행 조건을 검사합니다.</li>
  <li>완료, 재시도, 대기 또는 지정 Sequence로의 전환을 결정합니다.</li>
  <li>Process 완료 이벤트를 전달하고 Task의 실행 목록에서 제거합니다.</li>
</ol>

## 직접 개발한 기능

### Task와 Process 관리

- Task별 실행 Thread와 실행 주기 관리
- Process 추가·삭제 요청의 Queue 처리
- Pause, Resume, Abort 상태 처리
- 실행 전 PreCheck Sequence 지원
- 완료된 Sequence의 Listener 해제
- 현재 작업 조건에 맞는 Sequence로 Redirect

### Sequence 실행

- 정수 Step 기반 실행 위치 관리
- Delay와 경과 시간 확인
- Event Listener 등록과 해제
- 재시도와 완료 조건 검사
- 재실행 시 내부 상태와 Listener 초기화
- PreCheck의 Hold·Pass 상태와 일반 완료 상태 분리

### 이벤트 전달

- Event Class별 Dispatcher 생성
- 동일 Listener Name의 중복 등록 방지
- `CopyOnWriteArrayList` 기반 Listener 관리
- 동기 호출과 고정 Thread Pool 기반 비동기 호출
- 한 Listener의 실행 오류가 다른 Listener 호출에 영향을 주지 않도록 예외 분리

### 상태 머신 확장

- Enum 기반 상태 정의
- 상태별 진입·실행·종료 Action
- 조건에 따른 Transition 등록
- 상태 변경 Listener 제공

상태 머신은 프레임워크에 포함된 별도 확장 기능입니다. AYTSS의 차량 작업은 주로 `Task → Process → Sequence` 구조를 사용합니다.

## AYTSS 적용 범위

AYTSS에서는 TruckJob의 작업 종류와 차량 적재 상태를 기준으로 Process를 생성합니다. 각 Process는 대기구역, 트위스트락 작업구역, 안벽크레인과 야드블록 구간에 해당하는 Sequence를 조합합니다.

| 구분 | AYTSS 구현 수 |
|---|---:|
| Task | 4 |
| Process | 19 |
| Sequence | 27 |
| EventHandler | 28 |

작업 상태의 영속 저장은 AYTSS에서 담당합니다. WorkOrder 진행률, 현재 Sequence와 Alarm을 DB에 저장하고, 재기동 시 복원한 값으로 Process를 다시 구성합니다.

## 기술 구성

- Java 8
- 자체 EventHub와 Dispatcher
- Java Thread와 고정 크기 Thread Pool
- `CopyOnWriteArrayList` 기반 Listener 저장
- SLF4J 로깅
- AYTSS에서 라이브러리 JAR로 참조

## 관련 페이지

- [AYTSS 신규 구축]({{ '/projects/aytss/' | relative_url }})
- [eventflow 구현 기록]({{ '/2026/07/28/event-driven-workflow-boundaries.html' | relative_url }})

<p class="scope-note">원본 소스에는 사내 시스템과 연결되는 구현이 포함되어 있어 공개하지 않습니다. 이 페이지에는 코드에서 확인한 실행 구조, 직접 개발한 기능과 AYTSS 적용 범위만 정리했습니다.</p>
