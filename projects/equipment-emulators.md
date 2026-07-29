---
title: ECS·STS 장비 제어 프로그램
description: 상위 자동화 시스템의 명령을 크레인 작업으로 실행하고, 하부 장비 제어 계층과 내장 에뮬레이션 기능을 함께 구성한 항만 장비 제어 프로그램입니다.
permalink: /projects/equipment-emulators/
period: 2024
category: 항만 크레인 장비 제어 및 연동 검증
role: 구조 설계 및 전체 구현
stack: C#, .NET Framework 4.7.2, WinForms, 비동기 TCP Socket, Spring.NET, iBATIS
---

## 프로젝트 개요

ECS와 STS 프로그램은 단순히 크레인의 움직임을 화면에 재현하는 에뮬레이터가 아닙니다. ATCSS·QMS와 같은 상위 자동화 시스템의 작업지시를 받아 장비 상태와 실행 조건을 확인하고, 크레인 작업 Sequence를 수행한 뒤 진행·완료 상태를 보고하는 장비 제어 프로그램으로 구성했습니다.

장비 통신, 작업 관리, 상태 모델, 인터록, 자동·수동 운전과 제어 계층을 공통 구조로 두고, 하부 제어 계층에서 내장 에뮬레이션 기능을 선택할 수 있도록 분리했습니다.

<div class="compact-process" aria-label="ECS와 STS 장비 제어 프로그램의 실행 구조">
  <span>ATCSS·QMS<br>작업·제어 명령</span><b>→</b><span>ECS·STS<br>작업 Sequence</span><b>→</b><span>상태·인터록<br>장비 제어 계층</span><b>→</b><span>내장 에뮬레이션<br>또는 하부 장비 연동</span>
</div>

## 공통 제어 구조

| 영역 | 구현 내용 |
|---|---|
| 상위 통신 | 비동기 TCP 연결, 복수 장비 Session, 메시지 송수신과 재접속 |
| 프로토콜 | 현장별 메시지 변환·파싱, 명령 처리와 상태 응답 |
| 장비 모델 | 크레인 축, 위치, 운전 모드, 스프레더와 컨테이너 상태 관리 |
| 작업 실행 | 작업 수락·거절, 선행 조건, 실행 Sequence, 중단과 완료 처리 |
| 안전 조건 | 목표 위치와 작업 가능 상태 확인, 장비 간 작업 영역 간섭 검사 |
| 운전 화면 | 장비 상태와 작업 조회, 자동·수동 운전 및 개별 동작 확인 |
| 하부 제어 | 동일한 작업 Sequence에서 에뮬레이션 또는 PLC·장비 라이브러리 연동이 가능한 제어 경계 |

## ECS 야드 크레인 제어

ECS는 야드 크레인의 작업 명령을 실제 장비 실행 단위로 처리하도록 구성했습니다.

- 주행·횡행·권상 축과 스프레더 제어
- 블록·베이·열·단 위치와 좌표 변환
- 컨테이너 Pickup·Setdown과 적재 상태 관리
- 자동·수동 운전, 정지·비상정지와 작업 가능 조건
- 작업 수락, 진행, 완료, 거절과 중단 보고
- 복수 크레인의 개별 통신 Session과 작업 영역 간섭 확인

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/ecs-emulator.png' | relative_url }}" alt="ECS 야드 크레인 장비 제어 프로그램의 내장 에뮬레이션 화면">
  <figcaption>ECS 장비 제어 프로그램 — 복수 야드 크레인의 연결·운전 상태, 블록 위치, 작업 목록과 수동 축·스프레더 제어 화면</figcaption>
</figure>

## STS 안벽 크레인 제어

STS는 선박과 육상 사이의 컨테이너 양적하 작업을 크레인 실행 단위로 처리하도록 구성했습니다.

- Gantry, Main·Portal Trolley, Hoist와 Spreader 상태 관리
- Single·Dual Trolley 작업과 Platform 인계
- 선박·Platform·차량 사이의 컨테이너 이동
- 작업 순서, 차량 도착과 실행 가능 조건 확인
- 크레인 위치, 작업 단계와 완료 상태 보고
- 대기 작업에서 다음 실행 대상을 선정하는 Scheduling

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/sts-emulator.png' | relative_url }}" alt="STS 안벽 크레인 장비 제어 프로그램의 내장 에뮬레이션 화면">
  <figcaption>STS 장비 제어 프로그램 — 복수 안벽 크레인의 상태, 선박 Bay 배치, 크레인 측면·상면 위치와 수동 이동 제어 화면</figcaption>
</figure>

## 내장 에뮬레이션과 실장비 연동 구조

내장 에뮬레이션은 별도의 단순 모형이 아니라 ECS·STS의 작업 Sequence와 제어 명령을 그대로 사용합니다. 하부 장비 대신 소프트웨어에서 축 위치, 스프레더와 컨테이너 상태를 변경하고 그 결과를 동일한 상태 보고 경로로 전달합니다.

실장비 연동 시에는 이 하부 제어 영역에 PLC 및 장비별 통신 라이브러리를 연결하도록 경계를 분리했습니다. 따라서 상위 통신, 작업 Sequence, 상태 관리와 인터록 구조를 유지하면서 장비 명령 전송과 실제 상태 수신 부분을 현장 장비에 맞게 구현할 수 있습니다.

현재 프로젝트에 보관된 구현 범위에는 PLC 제조사별 드라이버가 포함되어 있지 않습니다. 실장비 운영에는 장비 라이브러리 연동과 현장별 I/O·안전 조건 검증이 추가로 필요합니다.

### TLS QMS 연동 검증

STS의 내장 에뮬레이션 기능을 [TLS QMS]({{ '/projects/tls-qms/' | relative_url }})와 연결해 작업지시 수신, Single/Dual Trolley 실행, 크레인 위치 변화와 진행·완료 상태 보고 흐름을 실제 크레인 없이 확인했습니다.

## 연동 검증 도구

- RFID 입력 에뮬레이터
- 차단기 및 주변 설비 상태 제어
- 센서 좌표 보정 도구
- 외부 인터페이스 송수신 확인 도구
- 장비별 연결 상태와 메시지 로그 화면

## 기술 스택

`C#`, `.NET Framework 4.7.2`, `WinForms`, `비동기 TCP Socket`, `Spring.NET`, `iBATIS`

<p class="scope-note">PLC 제조사별 드라이버, 메시지 규격, 안전 제어 설정과 고객·현장 식별 정보는 공개 범위에서 제외했습니다.</p>
