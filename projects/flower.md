---
title: Flower 오픈소스
description: Java 애플리케이션의 장시간 작업을 Flow와 Step으로 구성하고 실행·대기·복구·관측할 수 있게 만든 단일 JVM 오케스트레이션 런타임입니다.
permalink: /projects/flower/
period: 2026 - 현재
category: Java 오픈소스 프레임워크
role: 프로젝트 설계·개발, 테스트, 문서화 및 Maven Central 배포
stack: Java 8/17, Maven, Spring Boot, JDBC, JUnit 5, Micrometer, OpenTelemetry
---

## 프로젝트 개요

Flower는 Spring 및 Java 애플리케이션 내부의 장시간 작업을 명시적인 실행 단위로 구성하는 오픈소스 런타임입니다.

`Engine → Worker → Flow → Step → StepResult`

하나의 Flow는 업무 인스턴스 하나를 나타내고, Step은 해당 업무의 실행 단계를 나타냅니다. 각 Step은 실행 결과로 유지, 완료, 재실행, 이동, 종료 또는 실패를 반환합니다.

| 항목 | 내용 |
|---|---|
| 공개 저장소 | [github.com/flowerjvm/flower](https://github.com/flowerjvm/flower) |
| 공개 버전 | 0.1.1 |
| 배포 | [Maven Central](https://central.sonatype.com/artifact/io.github.flowerjvm/flower-core/0.1.1) |
| 라이선스 | Apache License 2.0 |
| 호환성 | Core Java 8, Spring Boot Starter Java 17 |

## 직접 개발한 범위

- Engine, Worker, Flow, Step과 명시적 실행 결과 모델 설계
- 이벤트, Signal과 Timeout을 이용한 비동기 대기 처리
- Step 진입 전 Hold·Redirect·Fail을 처리하는 Guard
- Flow 중복 제출 정책과 취소·실패·완료 처리
- Flow 위치를 저장하고 재기동 후 복원하는 Checkpoint/Resume
- Spring Boot 자동 설정과 내부 상태 조회용 Dump·Console
- JDBC 체크포인트 저장소와 DB별 Schema
- 로그, Micrometer Metric과 OpenTelemetry 연동
- 수동 Clock과 Tick 기반 결정론적 테스트 도구
- 잘못된 Flower 사용 패턴을 검사하는 `flower-check`
- Callback·승인·도구 응답·Deadline 대기용 별도 Event-loop 실행 모델
- Maven Central 릴리스와 GitHub Actions 배포 구성

## 실행 모델

### Engine

- Clock, EventBus와 Worker 수명주기 관리
- 실행 중인 Worker와 Flow 상태 조회
- Listener를 통한 실행 상태 전달

### Worker

- 단일 Scheduler Thread에서 등록된 Flow를 주기적으로 실행
- Submit과 Cancel 요청을 Queue로 처리
- `tickOnce()`를 이용한 수동 실행과 테스트 지원

### Flow

- `flowType`과 `flowKey`로 실행 인스턴스 식별
- 문자열 Step ID를 이용한 실행 위치 관리
- 중복 제출 시 Reject, Ignore, Replace 정책 지원
- 실행 주체와 추적 정보를 위한 ExecutionContext 제공

### Step

- `onEnter`, `onTick`, `onExit`, `onReset` 수명주기
- Event 구독과 Step 종료 시 Listener 자동 해제
- Step 내부의 작은 실행 위치를 위한 `stepNo`
- `stay`, `done`, `repeat`, `goTo`, `finish`, `fail` 결과 처리

## 영속화와 복구

Flower의 영속 기능은 실행 전체를 재현하는 Replay가 아니라 현재 Flow 위치를 저장하는 Checkpoint/Resume 방식입니다.

- 현재 Step ID와 Index, `stepNo`, 실행 문맥과 정의 버전 저장
- 재기동 시 새로운 Flow 인스턴스를 생성하고 저장 위치에서 재개
- Step별 복구 정책 지정
- PostgreSQL, MySQL, Oracle, H2와 SQLite Schema 제공
- Core Flow와 Event-loop Flow의 저장소 분리

외부 API 호출이나 DB 변경은 애플리케이션에서 멱등성을 보장해야 하며, 다중 JVM 간 Flow 소유권과 분산 잠금은 Flower가 처리하지 않습니다.

## 공개 모듈

| 모듈 | 역할 |
|---|---|
| `flower-core` | Engine, Worker, Flow, Step, EventBus와 Clock |
| `flower-spring-boot-starter` | Spring Boot 자동 설정과 관리용 Dump·Console |
| `flower-persistence-jdbc` | Core Flow의 JDBC Checkpoint Store |
| `flower-eventloop` | Event·Signal·Approval·Deadline 중심 실행 모델 |
| `flower-eventloop-persistence-jdbc` | Event-loop Checkpoint Store |
| `flower-observability` | Logging, Metric, Tracing과 실행 완료 대기 |
| `flower-testkit` | ManualClock, 수동 Tick과 Flow Assertion |
| `flower-check` | Flower 사용 규칙 정적 검사 |
| `flower-check-maven-plugin` | Maven Verify 연동 |
| `flower-check-gradle-plugin` | Gradle Check 연동 |

## 적용 범위와 제한

- Spring 애플리케이션 내부의 다단계 작업 실행
- 이벤트·시간·외부 응답을 기다리는 비동기 업무
- 재시도, Timeout, 운영자 개입과 실행 상태 확인이 필요한 작업
- 단일 JVM에서 실행되는 소규모·중간 규모 오케스트레이션

Flower는 BPMN 엔진, 분산 Scheduler, Temporal 또는 분산 Saga 엔진을 대체하지 않습니다. 서비스 간 분산 트랜잭션이나 여러 노드가 공동 소유하는 실행이 필요하면 별도 분산 실행 도구가 필요합니다.

## 관련 페이지

- [Flower 실행 구조 기술 기록]({{ '/notes/flower-in-jvm-flow-runtime/' | relative_url }})
- [eventflow 실행 프레임워크]({{ '/projects/eventflow/' | relative_url }})

<p class="scope-note">저장소, 소스, 테스트, 문서와 릴리스 이력은 공개 GitHub 저장소에서 확인할 수 있습니다. 페이지의 버전과 모듈 정보는 Flower 0.1.1 공개 릴리스를 기준으로 작성했습니다.</p>
