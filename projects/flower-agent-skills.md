---
title: Flower Agent Skills
description: AI 코딩 에이전트가 Flower와 Flower Action Runtime을 올바르게 적용하도록 설계한 두 개의 공개 개발 가이드입니다.
permalink: /projects/flower-agent-skills/
period: 2026 - 현재
category: AI 코딩 에이전트 개발 가이드 · 오픈소스
role: 가이드 설계·작성, 규칙 정의, 설치 도구와 버전 관리
stack: Agent Skills, Markdown, Codex, Claude Code, Gemini CLI, Flower
---

## 프로젝트 개요

Flower Agent Skills는 AI 코딩 에이전트가 Flower 기반 애플리케이션 코드를 작성하거나 검토할 때 사용하는 공개 가이드입니다. 런타임 라이브러리가 아니라 설계 규칙, 구현 순서, 금지 패턴과 검증 절차를 `SKILL.md`와 주제별 참고 문서로 제공합니다.

| 스킬 | 공개 버전 | 적용 대상 | 저장소 |
|---|---:|---|---|
| `flower-app-guide` | 0.2.4 | Flower 0.1.1 기반 애플리케이션 | [flowerjvm/flower-agent-skills](https://github.com/flowerjvm/flower-agent-skills) |
| `flower-action-runtime-guide` | 0.3.2 | Flower Action Runtime 0.3.1 | [flowerjvm/flower-action-runtime-guide](https://github.com/flowerjvm/flower-action-runtime-guide) |

두 저장소는 Apache License 2.0으로 공개했습니다.

## 직접 개발한 범위

- 작업 유형에 따라 필요한 문서를 선택하는 `SKILL.md` 라우팅 구성
- 반드시 지켜야 하는 실행·안전 규칙과 구현 순서 정의
- 빠른 규칙, 이벤트·대기, 영속화, 통합, 테스트와 검증 문서 분리
- 가이드 버전과 대상 런타임 버전을 별도로 관리하는 버전 정책
- 공개 릴리스와 실제 소스를 기준으로 확인하는 검증 절차
- `flower-app-guide`의 Codex·Claude Code·Gemini CLI 설치 및 제거 스크립트
- Flower와 Action Runtime을 함께 사용할 때 두 스킬을 조합하는 기준

## flower-app-guide

Flower를 사용하는 호스트 애플리케이션의 Flow와 Step을 설계·구현·검토할 때 적용합니다.

- Flow와 Step의 책임 및 크기 결정
- Worker Lane 선택과 Tick 내부 Blocking 금지
- Step Guard를 이용한 진입 전 검사·대기·이동·실패 처리
- Event, Signal, Timeout과 외부 응답 대기
- Spring Boot, Kafka, Domain Event 연동 경계
- Checkpoint/Resume 기반 복구와 Side Effect 멱등성
- `flower-testkit` 기반 결정론적 테스트
- 호스트 프로젝트의 `flower-check` 도입과 빌드 검증

## flower-action-runtime-guide

업무 변경 Action의 등록, 정책, 승인과 실행 수명주기를 구현·검토할 때 적용합니다.

- `ActionProposal`과 `ActionDefinition` 설계
- Registry, Validation, Policy, Approval과 Pre-execution Guard
- Synchronous, Async와 Deferred Executor 선택
- `ActionRun`, `RunStore`, JDBC 영속화와 Version CAS
- 완료·취소 경합, 재기동 복구와 외부 Callback 검증
- Direct, Flower Workflow와 Flower Event-loop Backend 선택
- 0.1·0.2 버전에서 0.3.1로 이전할 때의 변경 기준
- Backend 간 동작 일치, 실제 동시성과 복구 테스트

## flower-check와의 역할 구분

| 도구 | 역할 |
|---|---|
| Agent Skills | 구현 전에 구조와 규칙을 제공하고 작성·검토 절차를 안내 |
| `flower-check` | 작성된 Java 소스에서 금지된 Flower 사용 패턴을 검사하고 빌드 차단 |
| `flower-testkit` | Flow 전환, Timeout과 복구 동작을 결정론적으로 검증 |

Agent Skills는 코딩 에이전트의 판단을 돕는 문서이고 `flower-check`는 소스에 적용되는 정적 검사기입니다. 두 도구는 대체 관계가 아니라 작성 지침과 자동 검증 단계로 함께 사용합니다.

## 관련 페이지

- [Flower 오픈소스]({{ '/projects/flower/' | relative_url }})
- [Flower Action Runtime]({{ '/projects/flower-action-runtime/' | relative_url }})
- [Flower 코드 품질 도구 기술 기록]({{ '/notes/flower-check-and-agent-skills/' | relative_url }})
- [Flower JVM 프로젝트 역할 구분]({{ '/notes/flower-jvm-project-boundaries/' | relative_url }})
