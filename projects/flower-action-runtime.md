---
title: Flower Action Runtime
description: UI·API·Batch·MCP·AI가 요청한 업무 Action을 Registry·검증·정책·승인·중복 방지·감사 절차를 거쳐 실행하는 Java 통제 런타임입니다.
permalink: /projects/flower-action-runtime/
period: 2026 - 현재
category: 업무 Action 통제 런타임 · 오픈소스
role: 프로젝트 설계·개발, 영속화·동시성 제어, 테스트·문서화 및 Maven Central 배포
stack: Java 21, Maven, Flower, JDBC, CAS, JUnit 5
---

## 프로젝트 개요

Flower Action Runtime은 업무 변경을 요청하는 진입점과 실제 Domain Service 사이에 하나의 통제 경계를 제공하는 Java 런타임입니다. 사용자, UI, REST API, Batch, Scheduler, MCP와 AI Planner의 요청을 모두 `ActionProposal`로 변환해 동일한 절차로 처리합니다.

| 항목 | 내용 |
|---|---|
| 공개 저장소 | [github.com/flowerjvm/flower-action-runtime](https://github.com/flowerjvm/flower-action-runtime) |
| 공개 버전 | 0.3.1 |
| 배포 | [Maven Central](https://central.sonatype.com/artifact/io.github.flowerjvm/flower-action-runtime-core/0.3.1) |
| 라이선스 | Apache License 2.0 |
| 기반 | Java 21, Flower 0.1.1 |

## 통제 Pipeline

<ol class="flow-list">
  <li>ActionProposal과 실행 주체·Tenant·Run ID를 기록합니다.</li>
  <li>Action Registry에서 등록된 실행 정의를 조회합니다.</li>
  <li>입력값과 Action Definition을 검증합니다.</li>
  <li>Request Channel, Proposer Type, 실행 권한과 위험도를 기준으로 정책을 평가합니다.</li>
  <li>Idempotency Key를 기준으로 중복 실행을 예약하거나 기존 결과를 반환합니다.</li>
  <li>필요한 작업은 승인 대기 상태로 전환합니다.</li>
  <li>실행 직전에 정책을 재평가하고 Pre-execution Guard를 수행합니다.</li>
  <li>Executor를 호출하고 ActionRun 상태와 감사 결과를 저장합니다.</li>
</ol>

## 직접 개발한 기능

### Action 모델

- 안정적인 Action ID와 `ActionDefinition`
- Read·Write 등 Effect와 Risk Level
- 허용 Request Channel과 Proposer Type
- 실행 주체, Tenant, Run ID와 Idempotency Key 분리
- 기계 판정을 위한 Result Code와 Retry Disposition

### 정책과 승인

- 등록되지 않은 Action 실행 차단
- Input Validator와 Policy Gate
- 위험도 및 AI 제안 여부에 따른 승인 요구
- 승인·거절·만료 상태와 이후 Resume
- 승인 이후 Definition·Input·Policy 재검증
- 실행 직전 현재 업무 상태를 확인하는 Pre-execution Guard

### 실행 방식

| 방식 | 용도 |
|---|---|
| Synchronous | 호출 범위 안에서 끝나는 짧은 작업 |
| Async | 현재 프로세스가 완료를 소유하는 비동기 작업 |
| Deferred | Queue·원격 Worker·Callback이 완료를 담당하는 장시간 작업 |

Deferred 작업은 외부 Operation ID와 Attempt Token을 저장합니다. 완료 Callback과 취소 요청은 현재 Attempt를 확인하고, 이미 종료된 Run에 대한 중복 요청은 저장된 최종 결과를 반환합니다.

### 영속화와 동시성

- `ActionRun` 기반 실행 수명주기 저장
- 승인 대기, 외부 완료 대기와 최종 결과 조회
- JDBC `RunStore`
- Version 기반 Compare-and-set 상태 변경
- 완료와 취소가 동시에 발생할 때 하나의 최종 상태만 허용
- 재기동 후 비종료 Run 조회와 복구 지점
- 감사 이벤트와 Trace 정보 기록

## Flower 실행 Backend

| 모듈 | 역할 |
|---|---|
| `flower-action-runtime-core` | 공통 Pipeline과 Direct Runtime |
| `flower-action-runtime-workflow` | 같은 Pipeline을 Flower Flow·Step으로 실행 |
| `flower-action-runtime-persistence-jdbc` | JDBC ActionRun 저장 |
| `flower-action-runtime-eventloop` | 승인·외부 응답·Deadline 대기 |
| `flower-action-runtime-integration-test` | 모듈 간 복구·동시성 검증 |

Direct와 Workflow Backend는 동일한 Pipeline을 사용하며, 정책 결과가 달라지지 않도록 Parity Test로 검증합니다.

## 적용 범위와 제한

Flower Action Runtime은 업무 Side Effect의 실행 허용 여부와 수명주기를 관리합니다. AI Agent Framework, Model Client, MCP Server, BPMN Engine 또는 범용 Rules Engine이 아닙니다.

CAS는 Run 상태의 충돌을 막지만 외부 Side Effect의 Exactly-once 실행, 분산 Worker Lease와 Queue 전달을 보장하지 않습니다. 이 영역은 사용하는 애플리케이션에서 Idempotency, Outbox 또는 Worker Claim 방식으로 처리해야 합니다.

## 관련 페이지

- [Flower 오픈소스]({{ '/projects/flower/' | relative_url }})
- [Flower AI Harness]({{ '/projects/flower-ai-harness/' | relative_url }})
- [ArchDox]({{ '/projects/archdox/' | relative_url }})
- [Flower JVM 프로젝트 역할 구분]({{ '/notes/flower-jvm-project-boundaries/' | relative_url }})

