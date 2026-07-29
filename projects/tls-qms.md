---
title: TLS QMS
description: TOS의 선박 작업지시를 안벽 크레인 작업으로 연결하고 진행 상태를 관리하는 TLS의 Quay Management System입니다.
permalink: /projects/tls-qms/
period: 2026 · 설계 및 개발
category: 항만 안벽 크레인 작업 실행 · Java
role: QMS Java 서버 개발, STS ECS 연동과 에뮬레이터 검증
stack: Java 8 · Spring Boot · Oracle · TCP Socket
---

## 프로젝트 개요

TLS는 컨테이너 터미널의 여러 자동화 장비와 TOS 사이에서 작업 실행을 담당하는 시스템입니다. QMS(Quay Management System)는 그중 선박의 양하·적하 작업과 STS(Ship-to-Shore Crane) 연동을 담당합니다.

TOS에서 전달된 WorkQueue와 작업지시를 기준으로 크레인이 수행할 작업을 관리하고, STS ECS와 통신하면서 작업 시작부터 완료까지의 진행 상태를 처리합니다.

## 담당 범위

| 구분 | 내용 |
|---|---|
| 작업 관리 | 선박 WorkQueue와 WorkOrder 수신 및 실행 상태 관리 |
| 작업 실행 | 양하·적하 작업을 크레인이 처리할 실행 단위로 관리 |
| 크레인 대응 | Single Trolley와 Dual Trolley 작업 지원 |
| 장비 연동 | STS ECS 작업지시 전송과 진행 상태 수신 |
| 상태 제공 | 크레인 위치와 작업 진행·완료 상태를 관련 시스템에 제공 |

## QMS 작업 흐름

<div class="compact-process" aria-label="QMS의 작업 처리 흐름">
  <span>TOS<br>선박 작업지시</span><b>→</b><span>QMS<br>작업 실행 관리</span><b>→</b><span>STS ECS<br>크레인 작업</span><b>→</b><span>진행·완료<br>상태 반영</span>
</div>

QMS는 작업 순서와 실행 상태를 관리하고, 실제 크레인의 구동과 안전 제어는 하부 STS ECS가 담당합니다. Dual Trolley 장비에서는 두 트롤리의 작업 상태와 인계 과정을 고려해 선박과 차량 사이의 양적하 작업을 관리합니다.

## STS 에뮬레이터 연동

실제 STS 장비를 사용하지 않고도 QMS의 작업지시와 상태 처리 흐름을 확인할 수 있도록 직접 개발한 STS 에뮬레이터를 연동했습니다.

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/sts-emulator.png' | relative_url }}" alt="TLS QMS 연동 검증에 사용한 STS 에뮬레이터 실행 화면">
  <figcaption>STS 에뮬레이터 — 복수 STS 상태, 선박 Bay와 Container 배치, 크레인 위치와 작업 진행 확인 화면</figcaption>
</figure>

<div class="compact-process" aria-label="QMS와 STS 에뮬레이터 연동 흐름">
  <span>QMS<br>작업지시</span><b>→</b><span>STS Emulator<br>명령 수신</span><b>→</b><span>크레인 작업<br>상태 재현</span><b>→</b><span>진행 결과<br>QMS 보고</span>
</div>

에뮬레이터를 통해 Single/Dual Trolley 작업, 크레인 위치 변화, 컨테이너 이동과 작업 진행·완료 보고를 실제 장비 없이 확인했습니다.

<p class="scope-note">회사 내부 아키텍처, 메시지 규격, 현장 설정과 고객 식별 정보는 공개 범위에서 제외했습니다.</p>
