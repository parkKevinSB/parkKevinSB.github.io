---
title: AYTSS의 TOS 작업지시 처리
description: TOS TruckJob을 FMS WorkOrder로 변환하고 차량 위치와 진행 상태에 따라 실행하는 구조를 정리합니다.
tags: [AYTSS, 항만 자동화, Spring Boot]
reading_time: 6
date: 2026-07-28 17:35:00 +0900
---

## 시스템 구성

AYTSS는 TOS와 FMS 사이에서 자율 야드 이송장비의 작업을 생성하고 진행 상태를 관리합니다.

<!--more-->

| 연계 대상 | 처리 내용 |
|---|---|
| TOS | TruckJob, 장비, 선박과 크레인 정보 수신 |
| AYTSS | WorkOrder 생성, 작업 유형 결정, 실행 상태 관리 |
| FMS | 차량 Task 전송, 위치와 Progress 수신 |
| 크레인 자동화 | 야드블록 작업 진행 정보 연계 |
| DB | WorkOrder, Sequence, Alarm과 컨테이너 상태 저장 |

## TruckJob 수신

TOS 메시지를 TruckJob 객체로 변환한 뒤 EventHub를 통해 Handler에 전달합니다.

- 작업 생성·변경·취소·완료 처리
- 기존 WorkOrder ID와 진행 상태 유지
- 차량 배정 변경
- 컨테이너와 선박 작업 정보 갱신
- 수동 작업 Progress 입력 처리

## WorkOrder 생성

차량별 WorkOrder 생성 Sequence가 다음 조건을 확인합니다.

- 시스템 초기화 완료
- TOS와 FMS 연결 상태
- 차량 사용 가능 여부와 운전 모드
- 차량 알람과 Pause 상태
- 기존 WorkOrder 존재 여부
- Pending 또는 Lift-on 완료 TruckJob
- 작업 대상 위치 유효성

조건을 만족하면 TruckJob을 조합해 WorkOrder를 만들고 FMS Task를 전송합니다. FMS ACK가 수신되면 생성 완료 Event를 발생시키며, 응답이 없으면 Process를 제거하고 실패 상태를 전달합니다.

## 작업 유형 결정

| TOS 작업 | WorkOrder 단계 | 실행 Process |
|---|---|---|
| 양하 | 공차 이동·상차 | Discharging Unladen |
| 양하 | 적재 이동·하차 | Discharging Laden |
| 적하 | 공차 이동·상차 | Loading Unladen |
| 적하 | 적재 이동·하차 | Loading Laden |
| 야드 이송 | 적재 | Yard Laden |
| 야드 이송 | 공차 | Yard Unladen |

## 구간별 Sequence

- PreCheck: WorkOrder와 차량 상태 확인
- Pre-Waiting Area: 사전 대기구역 도착과 이동 허가
- Waiting Area: 크레인·트위스트락 작업구역 진입 허가
- TwistLock Area: 트위스트락 작업 결과 처리
- Quay Crane: 안벽크레인 도착, 정렬과 상·하차
- Yard Block: 블록 진입, 장비 정렬과 상·하차
- Unknown Area: 현재 위치를 확인할 수 없는 상태 처리

## FMS Progress 반영

FMS에서 수신한 Progress에 따라 다음 정보를 갱신합니다.

- 차량의 현재·다음 Area
- WorkOrder 진행 상태
- 목적 크레인 종류
- 도착 Block과 Bay
- 정밀 정렬 시작·완료
- 첫 번째·두 번째 컨테이너 완료 상태

처리 결과에 따라 FMS에 ACK OK 또는 NG를 반환하고 변경된 WorkOrder를 저장합니다.

## 재기동 복구

시스템 시작 시 DB에서 진행 중 WorkOrder를 읽어 다음 상태를 복원합니다.

- WorkOrder Progress
- 현재 Sequence와 Step
- 차량 Alarm
- 완료된 컨테이너
- 차량 적재 상태

복원 후 차량에 Process 재생성 표시를 설정하고, WorkOrder 생성 Task가 기존 작업에 맞는 Process를 다시 구성합니다.

## 기술 스택

`Java 8`, `Spring Boot 2.7`, `Gradle`, `Spring JDBC`, `MyBatis`, `HikariCP`, `Oracle`, `TCP Socket`, `Lombok`, `Gson`

상세 프로젝트 정보는 [AYTSS 신규 구축]({{ '/projects/aytss/' | relative_url }})에 정리했습니다.
