---
title: eventflow 실행 구조
description: AYTSS에 적용한 Task·Process·Sequence·EventHub 기반 실행 프레임워크의 구성과 동작을 정리합니다.
tags: [eventflow, Java, 작업 실행]
reading_time: 5
date: 2026-07-28 17:20:00 +0900
---

## 개발 목적

장시간 실행되는 장비 작업을 메시지 Handler 내부에 직접 구현하지 않고, 실행 단위와 상태 관리 책임을 분리하기 위해 `eventflow`를 개발했습니다.

<!--more-->

## 핵심 구성

| 구성 | 역할 |
|---|---|
| EventHub | Event Type별 Listener 등록과 동기·비동기 전달 |
| EventHandler | 수신 Event를 검증하고 업무 상태에 반영 |
| Task | 독립 Thread와 실행 주기 관리 |
| Process | PreCheck와 Sequence Queue 관리 |
| Sequence | 명령 실행, Event 대기, 재시도와 완료 판단 |
| TaskRegistry | Task 등록, 초기화, 시작·중지 |

## Task

- Task별 실행 Thread 생성
- 설정된 주기로 등록된 Process 실행
- Process 추가·삭제 요청을 Queue로 처리
- Pause, Resume, Abort 상태 관리
- Process 완료 Event 수신 후 목록에서 제거

## Process

- 실행 전 PreCheck Sequence 지원
- Sequence를 Queue 순서로 실행
- 완료된 Sequence의 Listener 해제
- `PASS` 상태의 Sequence를 뒤로 이동
- 지정한 Sequence로 Redirect
- 현재 Sequence 완료 후 다른 Sequence로 Redirect

## Sequence

- 정수 Step 기반 실행 위치 관리
- Delay와 경과 시간 확인
- Event Listener 등록과 해제
- 완료 조건 검사
- 재실행 시 내부 상태와 Listener 초기화
- PreCheck Hold·Pass와 일반 완료 상태 분리

## Event 처리

- Event Class별 Dispatcher 생성
- 동일 Listener Name의 중복 등록 방지
- CopyOnWriteArrayList 기반 Listener 관리
- 동기 전달과 고정 Thread Pool 기반 비동기 전달
- Listener 실행 오류를 다른 Listener와 분리

## AYTSS 적용

AYTSS에서는 다음 규모로 `eventflow` 구조를 사용합니다.

- Task 구현 4개
- Process 구현 19개
- Sequence 구현 27개
- EventHandler 구현 28개

TruckJob의 작업 종류와 적재 상태에 따라 Process를 생성하고, 대기구역·트위스트락 작업구역·안벽크레인·야드블록 Sequence를 조합합니다.

## 작업 복구

`eventflow` 자체가 영속 저장소를 제공하지는 않습니다. AYTSS에서 WorkOrder의 Progress, 현재 Sequence와 Alarm을 DB에 저장하고, 재기동 시 해당 값을 복원해 Process를 다시 생성합니다.

## 별도 StateMachine 기능

`eventflow` 라이브러리에는 Enum 기반 StateMachine과 조건부 Transition 기능도 포함되어 있습니다. AYTSS의 주요 작업 흐름은 이 기능보다 `Task → Process → Sequence` 구조를 사용합니다.

프레임워크 전체 구성은 [eventflow 실행 프레임워크]({{ '/projects/eventflow/' | relative_url }})에, 실제 적용 내용은 [AYTSS 신규 구축]({{ '/projects/aytss/' | relative_url }})에 정리했습니다.
