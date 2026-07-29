---
title: TLS QMS
description: TOS의 선박 작업계획과 WorkQueue·WorkOrder를 관리하고 STS ECS와 작업 명령·상태를 교환하는 TLS의 Quay Crane 실행 모듈입니다.
permalink: /projects/tls-qms/
period: 2026 · 설계 및 개발
category: TLS 안벽 크레인 작업 실행 · Java
role: QMS Java 서버 설계·개발, STS ECS 연동, Single/Dual Trolley Scheduling 및 검증
stack: Java 8 · Spring Boot · Spring Cloud Stream · RabbitMQ · TCP Socket · Oracle · MyBatis
---

## 프로젝트 개요

TLS는 TOS와 항만 장비 제어 시스템 사이에서 작업 실행을 담당하는 통합 실행 계층입니다. QMS(Quay Management System)는 그중 선박의 양하·적하 작업과 STS(Ship-to-Shore Crane) 실행을 담당하는 모듈입니다.

QMS는 TOS가 제공한 선박 정보와 WorkQueue·WorkOrder를 실행 상태로 관리하고, STS ECS에 선박·작업 정보 또는 트롤리별 Order를 전달합니다. STS가 보고한 위치·대기·작업 진행 상태는 QMS 도메인에 반영하고 다른 TLS 모듈과 TOS·GUI가 사용할 수 있는 Event와 API로 제공합니다.

| 구분 | QMS 처리 범위 |
|---|---|
| TOS 작업계획 | Vessel Visit, 선박 구조·적재 정보, WorkQueue와 WorkOrder 수신·조회·상태 관리 |
| 실행 상태 | WorkQueue·WorkOrder의 변경되지 않는 계획과 진행 중 Runtime 상태 분리 |
| STS 작업 | WorkQueue 전달 방식과 QMS 내부 Order 생성 방식을 설정으로 분리 |
| 장비 연동 | STS ECS TCP Session, 선박 정보 요청·응답, 작업 지시와 진행 보고 처리 |
| 외부 제공 | 장비 상태·위치·도착·작업 진행 Event 발행과 QMS 조회 API |
| 제어 경계 | QMS는 논리 작업 순서와 자원 상태를 관리하고 물리 충돌·안전 Interlock은 STS ECS·PLC가 담당 |

<div class="project-metrics" aria-label="QMS 코드 구성">
  <div><strong>2</strong><span>Dispatcher Mode</span></div>
  <div><strong>2</strong><span>Single / Dual Trolley</span></div>
  <div><strong>367</strong><span>Main Java File</span></div>
  <div><strong>34</strong><span>Java Test File</span></div>
</div>

<p class="scope-note">소스 수치는 2026년 7월 분석 시점 기준입니다. 실제 현장 전문, 고객 식별 정보, 접속 설정과 운영 데이터는 공개하지 않습니다.</p>

## TLS에서 QMS의 위치

<div class="architecture-diagram" role="img" aria-label="TLS 실행 계층에서 QMS의 위치">
  <div class="architecture-layer architecture-entry">
    <p class="diagram-label">상위 운영 시스템</p>
    <div class="architecture-items">
      <div><strong>TOS</strong><span>선박 작업계획 · WorkQueue · WorkOrder · 장비 상태</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>업무 지시와 상태 Event</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">TLS · Terminal Execution Layer</p>
    <div class="architecture-items architecture-items-3">
      <div class="diagram-focus"><strong>QMS</strong><span>Quay Crane 작업 실행 · STS 연동</span></div>
      <div><strong>Yard / Transfer</strong><span>ATCSS · AYTSS · FMS 실행 모듈</span></div>
      <div><strong>Common Services</strong><span>공통 데이터 · Event · Task · 통신 기반</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>장비별 실행 명령과 진행 보고</span></div>
  <div class="architecture-layer architecture-store">
    <p class="diagram-label">하부 장비 제어 시스템</p>
    <div class="architecture-items architecture-items-3">
      <div class="diagram-focus"><strong>STS ECS</strong><span>안벽 크레인 · Main/Portal Trolley · CraneOrder</span></div>
      <div><strong>Transfer Equipment</strong><span>AYT · AVG · AGV 등 이송장비</span></div>
      <div><strong>Yard Crane ECS</strong><span>야드 크레인 작업 실행과 장비 제어</span></div>
    </div>
  </div>
</div>

QMS가 STS 모듈을 담당하며, ATCSS·AYTSS·FMS 등 다른 실행 모듈의 전체 개발을 QMS 담당 범위로 포함하지는 않습니다.

## QMS 처리 구조

<div class="architecture-diagram" role="img" aria-label="QMS의 입력부터 작업 실행과 보고까지의 처리 구조">
  <div class="architecture-layer architecture-entry">
    <p class="diagram-label">외부 입력</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>TOS / IFB</strong><span>선박 · WorkQueue · 장비 상태</span></div>
      <div><strong>TMS</strong><span>차량 도착과 작업 정보</span></div>
      <div><strong>STS ECS</strong><span>위치 · 대기 · 작업 진행 보고</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>RabbitMQ · REST · TCP</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">입력 해석과 내부 Event</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>Adapter</strong><span>Broker · API · Socket 입력 경계</span></div>
      <div><strong>Interpreter</strong><span>Payload 해석 · 중복 입력 차단</span></div>
      <div><strong>Event Hub</strong><span>구체 Domain Event와 Handler 연결</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>도메인 상태 반영</span></div>
  <div class="architecture-layer architecture-primary">
    <p class="diagram-label">Plan / Runtime Domain</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>Vessel</strong><span>Visit · Geometry · Inventory</span></div>
      <div><strong>Work</strong><span>WorkQueue · WorkOrder · StsTask</span></div>
      <div><strong>Equipment</strong><span>Crane · Trolley · Platform · Truck</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>선택된 Dispatcher Mode</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">실행</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>ECS Dispatcher</strong><span>WorkQueue를 STS ECS에 전달하고 ECS가 CraneOrder 생성</span></div>
      <div><strong>QMS Dispatcher</strong><span>QMS Scheduler가 WorkOrder를 StsTask와 Order로 분해</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>STS 전문 송수신과 진행 반영</span></div>
  <div class="architecture-layer architecture-store">
    <p class="diagram-label">출력</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>STS Gateway</strong><span>대상 Session 선택 · 송수신 Queue · ACK/응답 연계</span></div>
      <div><strong>QMS Event / API</strong><span>장비 상태 · 위치 · 도착 · 작업 진행 조회와 발행</span></div>
    </div>
  </div>
</div>

## 두 가지 Dispatcher 방식

<div class="process-grid" aria-label="ECS Dispatcher와 QMS Dispatcher 비교">
  <article>
    <span>현재 ACTS 기본 구성</span>
    <h3>ECS Dispatcher</h3>
    <p>TOS에서 받은 WorkQueue와 선박 공통 정보를 단일 STS ECS Gateway에 전달합니다. 실제 크레인 선택, 작업 순서와 CraneOrder 생성은 STS ECS가 담당하고 QMS는 진행 보고를 WorkOrder 상태와 외부 Event로 변환합니다.</p>
  </article>
  <article>
    <span>QMS 내부 스케줄링 구성</span>
    <h3>QMS Dispatcher</h3>
    <p>활성 WorkQueue의 WorkOrder 후보를 Crane·Trolley별로 평가합니다. 실행 가능한 작업을 선택해 StsTask를 만들고 트롤리별 Order로 연결하는 방식입니다.</p>
  </article>
</div>

| 설정 축 | 값 | 의미 |
|---|---|---|
| 전송 대상 | `SINGLE_GATEWAY` | 하나의 STS ECS Gateway Session으로 선박과 작업 메시지 전송 |
| 전송 대상 | `PER_CRANE` | Crane별 Session을 찾아 개별 메시지 전송 |
| 작업 분배 | `ECS_DISPATCHER` | QMS는 WorkQueue를 전달하고 ECS가 실행 Order 생성 |
| 작업 분배 | `QMS_DISPATCHER` | QMS가 WorkOrder를 선택하고 Trolley별 StsTask·Order 생성 |

현재 ACTS 프로파일의 기본 조합은 `SINGLE_GATEWAY + ECS_DISPATCHER`입니다. `QMS_DISPATCHER`에는 Single/Dual Trolley Scheduler, 후보 제약·점수 계산, StsTask 생성, Order 메시지와 진행 보고 처리 구조가 구현되어 있으며 자동 배정부터 전체 완료까지의 현장 End-to-End 검증은 진행 범위로 구분했습니다.

## QMS Dispatcher의 작업 분해

<div class="compact-process" aria-label="QMS Dispatcher 작업 분해 순서">
  <span>활성 WorkQueue</span><b>→</b><span>WorkOrder 후보</span><b>→</b><span>제약 조건 평가</span><b>→</b><span>우선순위 점수</span><b>→</b><span>StsTask 생성</span><b>→</b><span>STS Order</span>
</div>

Scheduler는 실행 후보를 결정하고, Taskflow는 Crane·Trolley별 반복 실행과 상태 전환을 담당하도록 분리했습니다.

- `SchedulingTask`가 등록된 STS Crane을 기준으로 실행 Process를 구성합니다.
- Single Trolley는 `MAIN`, Dual Trolley는 `MAIN`과 `PORTAL` Lane을 각각 운영합니다.
- Pre-check에서 QMS 준비 상태, Dispatcher Mode, Trolley 이동·대기 상태, 기존 활성 Task와 WorkQueue 활성 여부를 확인합니다.
- Candidate는 WorkQueue·WorkOrder 순서, 현재 Bay와 목표 Bay 거리, Platform 점유를 점수로 계산합니다.
- Truck 도착, Platform 빈 Slot 또는 대상 Container, 선박 적재 물리 순서와 선적 작업 순서를 제약 조건으로 적용합니다.
- 선택 결과를 `StsTask`와 Runtime으로 분리 저장하고 WorkOrder·WorkQueue Runtime에 연결합니다.
- 반복 주기는 Worker Thread를 점유하는 대기 대신 Task Sequence의 지연 실행으로 처리합니다.

## Single / Dual Trolley 작업

Single Trolley는 선박과 차량 사이의 하나의 WorkOrder를 MAIN Trolley 작업으로 처리합니다. Dual Trolley는 Platform을 인계 지점으로 사용해 하나의 양적하 작업을 두 개의 실행 Leg로 분리합니다.

### 양하

<div class="compact-process" aria-label="Dual Trolley 양하 작업">
  <span>선박</span><b>→</b><span>MAIN Trolley<br>Ship to Platform</span><b>→</b><span>Platform</span><b>→</b><span>PORTAL Trolley<br>Platform to Truck</span><b>→</b><span>차량</span>
</div>

### 적하

<div class="compact-process" aria-label="Dual Trolley 적하 작업">
  <span>차량</span><b>→</b><span>PORTAL Trolley<br>Truck to Platform</span><b>→</b><span>Platform</span><b>→</b><span>MAIN Trolley<br>Platform to Ship</span><b>→</b><span>선박</span>
</div>

Platform Slot은 `EMPTY`, `RESERVED`, `OCCUPIED`, `BLOCKED`, `MAINTENANCE` 등의 상태로 관리합니다. QMS Scheduler는 Platform 수용량, 차량 도착, 반대편 Trolley의 선행 Leg 완료와 선박 적재 순서를 확인합니다. 실제 Trolley 간 충돌 회피와 안전 높이 같은 물리 제어는 STS ECS·PLC의 책임으로 유지합니다.

## STS ECS 통신

STS와 CPS 통신은 서로 다른 논리 Socket Runtime으로 분리하고 Netty Transport 자원은 공유하도록 구성했습니다. 각 Runtime은 Session·Endpoint·Protocol·Queue 상태를 독립적으로 관리합니다.

| 영역 | 구현 내용 |
|---|---|
| 연결 | 단일 Gateway 또는 Crane별 TCP Session 선택, 연결·재접속과 Lifecycle 관리 |
| 처리 | Netty I/O Thread는 Frame을 Queue에 넣고 업무 처리는 Taskflow의 Endpoint Lane에서 수행 |
| 안정성 | Session별 제한된 송수신 Queue와 Backpressure, 전송 실패 결과 확인 |
| 요청·응답 | Endpoint·Session·Header ID를 조합한 상관관계 등록, Timeout·취소·연결 종료 처리 |
| 프로토콜 | 선박 Visit·Geometry·Inventory, WorkQueue, Order, 진행·대기·위치·Platform 상태 전문 |

Socket Queue는 메모리 기반 `at-most-once` 전송 경계입니다. 업무 ACK, Timeout, Retry와 중복 방지는 메시지·Taskflow 계층에서 별도로 처리합니다.

## STS 에뮬레이터를 개발한 이유

QMS는 실제 STS ECS와 메시지를 주고받아야 전체 흐름을 확인할 수 있지만, 개발 기간 동안 운영 중인 안벽 크레인을 계속 사용할 수 없습니다. 그래서 QMS의 두 Dispatcher 방식과 Single/Dual Trolley 작업을 장비 없이 반복 검증할 수 있도록 STS 에뮬레이터를 직접 개발했습니다.

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/sts-emulator.png' | relative_url }}" alt="TLS QMS 연동 검증에 사용한 STS 에뮬레이터 실행 화면">
  <figcaption>STS 에뮬레이터 — 복수 STS 상태, 선박 Bay와 Container 배치, Crane 측면·상면 위치, 수동 이동과 작업 진행 확인 화면</figcaption>
</figure>

에뮬레이터는 QMS에서 전달한 선박 정보와 WorkQueue 또는 트롤리별 Order를 받아 같은 Crane 실행 Sequence로 연결합니다. Gantry·Trolley·Hoist·Spreader, 선박·Platform·차량의 Container 상태를 변경하고 작업 진행·대기·위치 보고를 다시 QMS에 전달합니다.

- Common 전문: Ship Visit, Geometry, Inventory 요청·응답
- WorkQueue 방식: WorkQueue와 WorkOrder를 받은 뒤 ECS 내부에서 CraneOrder 생성
- Order 방식: QMS가 만든 MAIN·PORTAL Trolley Order 실행
- Single/Dual Trolley, Platform 인계와 차량 도착 조건
- Pickup·Setdown, 안전 높이, 작업 거절·중단과 진행 보고
- 복수 Crane Session과 연결 단절·재접속 시나리오

## 직접 개발한 Java 영역

- Spring Boot 기반 QMS 서버 구조와 기동 초기화
- TOS·TMS·공통 Broker Adapter, Payload Interpreter와 내부 Domain Event 연결
- Vessel Visit·Geometry·Inventory와 WorkQueue·WorkOrder Plan/Runtime 모델
- STS Crane, MAIN/PORTAL Trolley, Platform, Truck 상태 모델
- `SINGLE_GATEWAY`·`PER_CRANE` 전송과 `ECS_DISPATCHER`·`QMS_DISPATCHER` 모드 분리
- WorkQueue Report·Activation과 Trolley별 Order 메시지 생성·전송
- Single/Dual Trolley Scheduler, Candidate·Constraint·Scoring 구조
- Crane·Trolley별 Scheduling Task·Process·Pre-check Sequence
- STS TCP Runtime, Session Routing, bounded Queue와 요청·응답 상관관계
- STS 진행·대기·위치·도착 보고의 Runtime 반영
- 장비·작업 상태 RabbitMQ Event와 QMS REST 조회 API
- Oracle·MyBatis 저장 계층과 Task Checkpoint 구성
- Scheduler·Broker·STS 메시지·API·복구 단위 테스트

## 기술 구성

| 구분 | 기술 |
|---|---|
| 언어·프레임워크 | Java 8, Spring Boot 2.6, Spring Framework |
| 메시지 | Spring Cloud Stream, RabbitMQ, Kafka Binder 구성 |
| 장비 통신 | Netty 기반 TCP Socket Starter, 비동기 Queue, 요청·응답 Correlation |
| 실행 구조 | LI Event Starter, LI Task Starter, Crane·Trolley별 Taskflow |
| 데이터 | Oracle, MyBatis, HikariCP, JDBC Checkpoint |
| 검증 | JUnit, QMS Test Harness, STS Emulator, CPS Middleware 연동 |

## 관련 프로젝트

- [STS 장비 에뮬레이터]({{ '/projects/equipment-emulators/' | relative_url }}) — QMS의 WorkQueue·Order·진행 보고 연동 검증
- [ATCSS]({{ '/projects/atcss/' | relative_url }}) — TOS와 Yard Crane ECS 사이의 작업 실행 모듈
- [AYTSS]({{ '/projects/aytss/' | relative_url }}) — TOS 작업지시와 FMS 사이의 이송장비 실행 모듈

<p class="scope-note">이 페이지는 QMS 저장소의 Java 소스, 테스트, 설계 문서와 STS 인터페이스 자료를 대조해 작성했습니다. TLS의 다른 모듈과 고객사 인터페이스 원문은 담당 범위 또는 공개 범위에 포함하지 않았습니다.</p>
