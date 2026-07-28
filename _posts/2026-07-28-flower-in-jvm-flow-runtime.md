---
title: Flower 단일 JVM 실행 구조
description: 공개 오픈소스 Flower의 Engine·Worker·Flow·Step 실행 모델과 이벤트 대기, 체크포인트, 테스트 구성을 정리합니다.
tags: [Flower, Java, 오픈소스]
reading_time: 6
date: 2026-07-28 18:00:00 +0900
---

## 개발 범위

Flower는 Java 애플리케이션 내부의 장시간 작업을 작은 Step으로 분리해 실행하는 오픈소스 런타임입니다. 공개 버전 0.1.1을 Maven Central에 배포했으며, Core와 선택형 모듈을 분리해 필요한 기능만 사용할 수 있도록 구성했습니다.

<!--more-->

## 실행 계층

| 계층 | 책임 |
|---|---|
| Engine | Clock, EventBus, Worker와 Listener 관리 |
| Worker | Flow 제출·취소 Queue와 Tick 실행 |
| Flow | 하나의 업무 인스턴스와 Step 순서 관리 |
| Step | 한 단계의 진입·실행·종료 및 대기 처리 |
| StepResult | 유지·완료·재실행·이동·종료·실패 결정 |

```text
Engine
  └─ Worker
       └─ Flow
            └─ Step
                 └─ StepResult
```

Worker는 설정된 주기로 각 Flow의 현재 Step을 한 번씩 실행합니다. 사용자 코드가 Worker Thread를 오래 점유하지 않도록 Step은 외부 작업을 시작하거나 현재 상태를 확인한 뒤 결과를 반환하는 구조로 제한했습니다.

## Step 수명주기와 전환

- `onEnter`: Step이 현재 실행 위치가 될 때 한 번 호출
- `onTick`: Worker 주기마다 호출
- `onExit`: 완료, 이동, 종료 또는 실패로 Step을 벗어날 때 호출
- `onReset`: 재실행 전에 호출

전환 결과는 다음과 같이 구분됩니다.

| 결과 | 처리 |
|---|---|
| `stay()` | 현재 Step 유지 |
| `done()` | 다음 Step 진행 |
| `repeat()` | 현재 Step 초기화 후 재실행 |
| `goTo(id)` | 지정한 Step으로 이동 |
| `finish()` | Flow 정상 종료 |
| `fail(cause)` | Flow 실패 종료 |

## 이벤트와 Timeout

Step은 `onEnter`에서 Event를 구독하고 `onTick`에서 Signal이나 Domain State를 확인할 수 있습니다. Step 종료, 재실행 또는 Flow 종료 시 해당 Step이 등록한 Listener를 해제합니다.

- Event 수신 Callback은 Signal만 기록
- 다음 Tick에서 완료 조건 확인
- Deadline 경과 시 Timeout 처리
- 실제 업무 상태는 애플리케이션 DB 또는 Domain Store에서 확인

이 구조는 Event Callback 안에서 Flow 상태를 직접 변경하거나 Worker Thread를 Blocking하는 방식을 피하기 위한 것입니다.

## Checkpoint와 복구

Durable Flow는 현재 실행 위치를 Checkpoint Store에 저장합니다.

- Flow ID와 현재 Step ID
- Step Index와 `stepNo`
- ExecutionContext
- Flow Definition Version
- 실행 상태와 갱신 시각

재기동 시 애플리케이션이 새 Flow를 구성한 후 Checkpoint를 적용합니다. 이는 실행 이력을 재생하는 방식이 아니므로 외부 Side Effect의 중복 방지는 애플리케이션이 담당합니다.

JDBC 모듈은 PostgreSQL, MySQL, Oracle, H2와 SQLite Dialect 및 Schema를 제공합니다.

## Spring Boot 구성

Starter는 다음 Bean과 수명주기를 자동 구성합니다.

- Clock
- EventBus
- Engine
- 설정에 선언된 Worker
- 선택형 JDBC Checkpoint Store
- 애플리케이션 시작·종료와 연동되는 Engine Lifecycle

운영 확인을 위해 Engine Dump API와 간단한 내부 Console을 선택적으로 활성화할 수 있습니다. 공개 Endpoint가 아니라 기존 Spring Security나 내부 네트워크로 보호하는 관리 기능입니다.

## 결정론적 테스트

`flower-testkit`은 수동 Clock, 수동 Tick, In-memory EventBus, Recording Listener와 Fake Checkpoint Store를 제공합니다. Scheduler나 실제 시간을 사용하지 않고 Step 이동, Timeout과 복구를 검증할 수 있습니다.

## flower-check 빌드 검사

`flower-check`는 Java 소스를 분석해 알려진 Flower 사용 규칙을 빌드 단계에서 검사합니다. 현재 19개 규칙을 구현했으며, 주요 검사 범위는 다음과 같습니다.

- Worker Tick 안의 Blocking 호출
- Step 내부의 LLM·Provider SDK 직접 호출
- 다른 Flow의 직접 구동과 잘못된 `goTo` 대상
- 대기 Step의 Timeout·취소 및 Durable Step의 복구 정책
- Event Callback의 상태 결정과 Engine·Worker 수명주기 소유
- Step ID 중복, Step 인스턴스 공유와 `ExecutionContext` 오용
- 반복 Scheduler 승인, Guard Side Effect와 EventStep 복구
- Action Registry·Policy·승인 절차를 우회하는 선택형 규칙

JavaParser 기반 검사 엔진은 Plain Text와 SARIF를 출력합니다. 규칙별 심각도, 사유가 포함된 Suppression과 Baseline 파일을 지원해 기존 프로젝트에도 단계적으로 적용할 수 있습니다.

Maven Plugin은 `verify`, Gradle Plugin은 `check` 단계에 연결됩니다. 실제 임시 호스트 프로젝트를 이용한 Maven Invoker와 Gradle TestKit으로 Plugin 동작을 검증하고, Flower 저장소 자체도 Reactor 빌드에서 같은 검사를 수행합니다.

구현 지침을 제공하는 두 Agent Skill과의 역할 구분은 [Flower 코드 품질 도구]({{ '/notes/flower-check-and-agent-skills/' | relative_url }})에 별도로 정리했습니다.

## Event-loop 실행 모델

`flower-eventloop`는 Tick 기반 Core와 분리된 실행 모듈입니다. Callback, Signal, 외부 도구 응답, 승인과 Deadline처럼 대기 중심인 작업을 대상으로 합니다.

Event-loop는 Core Worker를 대체하지 않습니다. 일정 주기로 상태를 확인하는 작업은 Core를, 특정 이벤트나 외부 응답이 도착할 때 실행을 재개하는 작업은 Event-loop를 선택할 수 있습니다.

## 공개 범위

- 저장소: [flowerjvm/flower](https://github.com/flowerjvm/flower)
- 릴리스: [Flower 0.1.1](https://github.com/flowerjvm/flower/releases/tag/v0.1.1)
- 배포: [Maven Central flower-core](https://central.sonatype.com/artifact/io.github.flowerjvm/flower-core/0.1.1)
- 라이선스: Apache License 2.0

전체 프로젝트 정보는 [Flower 오픈소스]({{ '/projects/flower/' | relative_url }}) 페이지에 정리했습니다.
