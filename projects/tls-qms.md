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

QMS는 TOS에서 선박 정보와 WorkQueue·WorkOrder를 받아 작업 상태를 관리합니다. 설정된 Dispatcher 방식에 따라 작업계획을 STS ECS로 전달하거나 QMS에서 트롤리별 Order를 생성하고, STS의 작업 진행 결과를 다시 TOS와 관련 모듈에 전달합니다.

| 구분 | 담당 기능 |
|---|---|
| 작업계획 | 선박 정보, WorkQueue와 WorkOrder 수신·관리 |
| 작업 실행 | 양하·적하 순서와 Single/Dual Trolley 작업 관리 |
| 장비 연동 | STS ECS에 WorkQueue 또는 Order 전달 |
| 상태 처리 | 위치·대기·작업 진행·완료 상태 반영 |
| 외부 연계 | QMS Event 발행과 조회 API 제공 |

## TLS에서 QMS의 위치

<div class="architecture-diagram" role="img" aria-label="TLS 실행 계층에서 QMS의 위치">
  <div class="architecture-layer architecture-entry">
    <p class="diagram-label">상위 운영 시스템</p>
    <div class="architecture-items">
      <div><strong>TOS</strong><span>선박 작업계획 · WorkQueue · WorkOrder</span></div>
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
      <div class="diagram-focus"><strong>STS ECS</strong><span>안벽 크레인 · Main/Portal Trolley</span></div>
      <div><strong>Transfer Equipment</strong><span>AYT · AVG · AGV 등 이송장비</span></div>
      <div><strong>Yard Crane ECS</strong><span>야드 크레인 작업 실행과 장비 제어</span></div>
    </div>
  </div>
</div>

QMS는 TLS 안에서 STS 작업을 담당합니다. 다른 Yard·Transfer 실행 모듈과 업무 상태를 연계하지만, QMS의 직접 담당 범위는 선박 작업과 안벽 크레인 실행입니다.

## QMS 작업 흐름

<div class="compact-process" aria-label="QMS의 작업 처리 흐름">
  <span>TOS<br>선박·WorkQueue</span><b>→</b><span>QMS<br>작업 상태 관리</span><b>→</b><span>WorkQueue 또는<br>Trolley Order</span><b>→</b><span>STS ECS<br>크레인 작업</span><b>→</b><span>진행·완료<br>상태 반영</span>
</div>

QMS는 WorkQueue 안의 WorkOrder를 실행 가능한 작업 단위로 관리하고, STS ECS의 진행 보고를 기준으로 작업 상태를 갱신합니다. 논리적인 작업 순서와 실행 상태는 QMS가 관리하고, 크레인의 물리 제어와 안전 Interlock은 STS ECS·PLC가 담당합니다.

## Dispatcher 방식

<div class="process-grid" aria-label="ECS Dispatcher와 QMS Dispatcher 비교">
  <article>
    <span>현재 ACTS 기본 구성</span>
    <h3>ECS Dispatcher</h3>
    <p>QMS가 WorkQueue와 선박 정보를 STS ECS에 전달합니다. 실제 크레인 작업 순서와 CraneOrder는 STS ECS가 생성합니다.</p>
  </article>
  <article>
    <span>QMS 내부 스케줄링 구성</span>
    <h3>QMS Dispatcher</h3>
    <p>QMS가 WorkOrder를 평가하고 Single/Dual Trolley별 실행 Order를 생성해 STS ECS에 전달합니다.</p>
  </article>
</div>

현재 ACTS 구성은 `ECS_DISPATCHER`를 기본으로 사용합니다. QMS 내부에는 별도의 `QMS_DISPATCHER` 구조도 구현해 두 가지 STS 연동 방식에 대응했습니다.

## Single / Dual Trolley 작업

Single Trolley는 선박과 차량 사이의 작업을 하나의 트롤리가 수행합니다. Dual Trolley는 Platform을 중간 인계 지점으로 사용해 MAIN과 PORTAL Trolley가 작업을 나누어 수행합니다.

### 양하

<div class="compact-process" aria-label="Dual Trolley 양하 작업">
  <span>선박</span><b>→</b><span>MAIN Trolley</span><b>→</b><span>Platform</span><b>→</b><span>PORTAL Trolley</span><b>→</b><span>차량</span>
</div>

### 적하

<div class="compact-process" aria-label="Dual Trolley 적하 작업">
  <span>차량</span><b>→</b><span>PORTAL Trolley</span><b>→</b><span>Platform</span><b>→</b><span>MAIN Trolley</span><b>→</b><span>선박</span>
</div>

QMS는 WorkQueue 순서, 차량 도착, Platform 상태와 Trolley 작업 상태를 기준으로 다음 실행 작업을 관리합니다.

## STS 에뮬레이터 연동

실제 STS 장비 없이 QMS와 STS ECS 사이의 작업 흐름을 확인하기 위해 직접 개발한 STS 에뮬레이터를 연동했습니다.

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/sts-emulator.png' | relative_url }}" alt="TLS QMS 연동 검증에 사용한 STS 에뮬레이터 실행 화면">
  <figcaption>STS 에뮬레이터 — 복수 STS 상태, 선박 Bay와 Container 배치, Crane 측면·상면 위치와 작업 진행 확인 화면</figcaption>
</figure>

<div class="compact-process" aria-label="QMS와 STS 에뮬레이터 연동 흐름">
  <span>Ship Visit<br>Geometry · Inventory</span><b>→</b><span>WorkQueue 또는<br>MAIN·PORTAL Order</span><b>→</b><span>CraneOrder<br>작업 실행</span><b>→</b><span>장비·Container<br>상태 변경</span><b>→</b><span>진행·대기·위치<br>QMS 보고</span>
</div>

WorkQueue 방식은 에뮬레이터 내부에서 CraneOrder를 만들고, Order 방식은 QMS가 생성한 MAIN·PORTAL Trolley Order를 실행합니다. 두 방식 모두 같은 작업 Sequence에서 Gantry·Trolley·Hoist·Spreader와 선박·Platform·차량 상태를 처리하며, Single/Dual Trolley와 작업 거절·중단·재접속 흐름을 확인할 수 있습니다.

<p class="scope-note">공개 페이지에는 시스템 구조와 담당 범위만 정리했으며 실제 현장 전문과 고객 식별 정보는 포함하지 않았습니다.</p>
