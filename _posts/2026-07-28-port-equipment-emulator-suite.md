---
title: ECS·STS 장비 제어 프로그램 주요 개발
description: 야드·안벽 크레인의 작업 실행과 상태 보고, 하부 장비 제어 경계와 내장 에뮬레이션 기능을 정리합니다.
tags: [항만 자동화, 장비 제어, C#]
reading_time: 5
date: 2026-07-28 16:45:00 +0900
---

## 개발 대상

ECS와 STS는 상위 자동화 시스템의 명령을 받아 야드·안벽 크레인의 작업을 실행하는 장비 제어 프로그램입니다. 실제 장비를 사용할 수 없는 개발 단계에서도 같은 작업 Sequence를 검증할 수 있도록 에뮬레이션 기능을 프로그램 내부에 포함했습니다.

| 프로그램 | 역할 |
|---|---|
| ECS 장비 제어 | 야드 크레인 작업 명령, 실행 Sequence와 상태 보고 |
| STS 장비 제어 | 안벽 크레인의 선박 양적하 작업과 Trolley Scheduling |
| 내장 에뮬레이션 | 하부 장비 동작과 상태 변화를 소프트웨어로 실행 |
| 연동 도구 | RFID, 차단기, 센서와 외부 Interface 검증 |

<!--more-->

## 공통 구조

장비 제어 프로그램은 다음 영역으로 분리했습니다.

1. 상위 시스템 연결과 메시지 송수신을 담당하는 통신 계층
2. 현장별 메시지 변환과 명령 처리를 담당하는 프로토콜 계층
3. 장비, 위치, 컨테이너와 작업 상태를 관리하는 도메인 계층
4. 선행 조건, 인터록과 작업 단계를 처리하는 Sequence 계층
5. 장비 명령과 상태 수신을 담당하는 하부 제어 계층
6. 상태 모니터링과 수동 제어를 제공하는 WinForms 화면

하부 제어 계층에서는 운전 설정에 따라 내장 에뮬레이션을 실행하거나 PLC·장비 라이브러리를 연결할 수 있도록 경계를 구분했습니다. 현재 구현에는 에뮬레이션 동작이 포함되어 있으며, 제조사별 PLC 드라이버는 별도 연동 영역입니다.

## ECS 야드 크레인 제어

- Gantry·Trolley·Hoist 축과 Spreader 제어
- 블록·베이·열·단 위치와 컨테이너 상태 관리
- 자동·수동 운전, 정지와 비상정지 상태
- 작업 수락·거절, 실행·중단과 완료 보고
- 복수 크레인 Session과 작업 영역 간섭 확인

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/ecs-emulator.png' | relative_url }}" alt="ECS 야드 크레인 장비 제어 프로그램의 내장 에뮬레이션 화면">
  <figcaption>ECS 장비 제어 프로그램의 장비 상태, 야드 위치와 수동 제어 화면</figcaption>
</figure>

## STS 안벽 크레인 제어

- Gantry, Main·Portal Trolley, Hoist와 Spreader 상태 관리
- 선박·Platform·차량 사이의 컨테이너 작업
- Single·Dual Trolley 작업과 실행 대상 Scheduling
- 작업 가능 조건, 위치 변화와 진행·완료 상태 보고

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/sts-emulator.png' | relative_url }}" alt="STS 안벽 크레인 장비 제어 프로그램의 내장 에뮬레이션 화면">
  <figcaption>STS 장비 제어 프로그램의 크레인 상태, 선박 Bay 배치와 수동 이동 제어 화면</figcaption>
</figure>

## 내장 에뮬레이션 활용

에뮬레이션 모드에서도 실제 운전과 동일한 상위 메시지 처리, 작업 Sequence, 인터록과 상태 보고 경로를 사용합니다. 이를 통해 정상 작업, 작업 거절·중단, 통신 단절·재접속과 복수 장비 연결을 실장비 투입 전에 검증했습니다.

## 기술 스택

`C#`, `.NET Framework 4.7.2`, `WinForms`, `비동기 TCP Socket`, `Spring.NET`, `iBATIS`

상세 프로젝트 정보는 [ECS·STS 장비 제어 프로그램]({{ '/projects/equipment-emulators/' | relative_url }})에 정리했습니다.
