---
title: Flower JVM 프로젝트와 개발 도구 역할 구분
description: Flower, Bloom, AI Harness, Action Runtime, Agent Skills, flower-check와 ArchDox의 역할을 구분합니다.
tags: [Flower JVM, Java, Architecture]
reading_time: 5
date: 2026-07-28 18:15:00 +0900
---

## 프로젝트별 책임

| 프로젝트 | 담당 영역 |
|---|---|
| Flower | 장시간 업무를 Flow와 Step으로 실행 |
| Bloom | JVM 내부 객체·모듈 사이의 Event 전달 |
| Flower AI Harness | AI 호출의 제출·검증·재시도·취소 수명주기 |
| Flower Action Runtime | 업무 Action의 정책·승인·중복 방지·감사와 실행 통제 |
| Flower Agent Skills | 코딩 에이전트에 Flower와 Action Runtime의 구현·검토 규칙 제공 |
| `flower-check` | Java 소스의 잘못된 Flower 사용 패턴을 빌드 단계에서 차단 |
| ArchDox | 위 구성요소를 문서 업무에 적용한 실제 Platform |

<!--more-->

## Flower

Flower는 실행 순서를 소유합니다.

```text
Engine → Worker → Flow → Step → StepResult
```

Flow의 현재 Step, Event 대기, Timeout, Retry, Checkpoint와 복구를 관리합니다. Domain Data와 업무 규칙은 사용하는 Application이 소유합니다.

## Bloom

Bloom은 Event 전달만 담당합니다.

```text
Publisher → EventBus → Subscriber
```

하나의 JVM 안에서 Event를 동기 또는 비동기로 전달합니다. Flow 상태, Retry, 영속화와 분산 전송은 담당하지 않습니다.

## Flower AI Harness

Flower AI Harness는 AI 업무 한 건의 실행 수명주기를 관리합니다.

```text
Prompt 준비
→ Provider 제출·조회
→ Structured Output 검증
→ Retry·Refine·Model Fallback
→ Finding 전달
```

OpenAI, Anthropic, Spring AI, OpenAI 호환 API와 외부 CLI Agent를 공통 Gateway 계약 뒤에 배치합니다. Flower Worker는 Provider 응답을 Blocking으로 기다리지 않고 제출과 상태 확인만 수행합니다.

## Flower Action Runtime

Flower Action Runtime은 실제 업무 변경을 실행해도 되는지 결정합니다.

```text
ActionProposal
→ Registry
→ Validation
→ Policy
→ Duplicate Control
→ Approval
→ Pre-execution Guard
→ Executor
→ ActionRun·Audit
```

AI Harness의 결과가 업무 Action을 제안할 수는 있지만 직접 Domain Service를 호출하지 않습니다. Action Runtime이 실행 주체, Tenant, 위험도와 승인 상태를 확인한 뒤 등록된 Executor만 호출합니다.

## Flower 개발 도구

Flower Agent Skills와 `flower-check`는 Runtime이 아니라 개발 단계에 적용하는 도구입니다.

```text
Agent Skills로 구현 규칙 확인
→ 애플리케이션 코드 작성
→ flower-check로 정적 검사
→ flower-testkit으로 실행 동작 검증
```

`flower-app-guide`는 Flow·Step·Guard, 이벤트 대기, 복구와 테스트를 다룹니다. `flower-action-runtime-guide`는 Action 정책·승인, 실행 방식, JDBC CAS와 복구를 다룹니다. `flower-check`는 작성된 소스를 19개 규칙으로 검사하고 Maven `verify` 또는 Gradle `check`에서 빌드를 차단합니다.

## ArchDox 적용

ArchDox는 다음과 같이 역할을 나눕니다.

| ArchDox 구성 | 적용 |
|---|---|
| `archdox-ai-harness` | 문서 검토와 QA 등 AI 판단 업무 |
| `archdox-worker` | Domain Action의 Policy·승인·Trace와 실행 |
| `archdox-agent` | 문서 변환, 사진, Artifact와 Storage 작업 |
| Flower | 문서 Job과 장시간 업무의 Step 실행 |
| Bloom | Application 내부 Event 전달 |

`archdox-agent`는 AI Agent가 아니라 문서·파일·Storage 작업을 실행하는 등록형 실행 Server입니다.

## 공개 저장소

- [Flower](https://github.com/flowerjvm/flower)
- [Bloom](https://github.com/flowerjvm/bloom)
- [Flower AI Harness](https://github.com/flowerjvm/flower-ai-harness)
- [Flower Action Runtime](https://github.com/flowerjvm/flower-action-runtime)
- [Flower Agent Skills](https://github.com/flowerjvm/flower-agent-skills)
- [Flower Action Runtime Guide](https://github.com/flowerjvm/flower-action-runtime-guide)
- [ArchDox](https://github.com/parkKevinSB/archdox)
