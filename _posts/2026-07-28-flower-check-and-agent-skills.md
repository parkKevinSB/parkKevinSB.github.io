---
title: "Flower 코드 품질 도구: Agent Skills와 flower-check"
description: Flower 코드 작성 지침을 제공하는 두 Agent Skill과 19개 규칙을 빌드에서 검사하는 flower-check의 역할과 구현 범위를 정리합니다.
tags: [Flower, 개발 도구, 정적 분석]
reading_time: 6
date: 2026-07-28 18:30:00 +0900
---

## 개발 도구 구성

Flower 기반 코드를 작성하고 검증하는 도구를 다음 세 단계로 구분했습니다.

| 단계 | 도구 | 역할 |
|---|---|---|
| 작성·검토 | `flower-app-guide`, `flower-action-runtime-guide` | 코딩 에이전트에 설계 규칙, 구현 순서와 검증 항목 제공 |
| 정적 검사 | `flower-check` | Java 소스의 금지 패턴을 검사하고 빌드 실패 처리 |
| 동작 검증 | `flower-testkit` | 수동 Clock과 Tick으로 Flow 전환·Timeout·복구 테스트 |

<!--more-->

## 두 개의 Agent Skill

| 스킬 | 버전 | 주요 범위 |
|---|---:|---|
| `flower-app-guide` | 0.2.4 | Flow·Step·Guard, Worker Lane, Event 대기, Checkpoint/Resume, 테스트와 `flower-check` 적용 |
| `flower-action-runtime-guide` | 0.3.2 | Action 정책·승인, 실행 방식, JDBC CAS, 완료·취소 경합, 복구와 Flower Backend |

각 스킬은 `SKILL.md`에서 작업 종류에 맞는 참고 문서를 선택하고, 빠른 규칙과 검증 문서는 항상 확인하도록 구성했습니다. 가이드 버전과 대상 런타임 버전을 분리해 예제와 규칙이 어떤 공개 릴리스를 기준으로 작성됐는지 명시합니다.

## flower-check

`flower-check`는 Flower 저장소에 포함된 빌드 단계 개발 도구입니다. 호스트 애플리케이션의 Java 소스를 분석하고 알려진 Flower 사용 규칙을 위반하면 파일과 행 번호를 출력합니다.

현재 구현은 19개 규칙으로 구성됩니다.

| 규칙 범위 | 검사 내용 |
|---|---|
| 001~005 | Worker Thread Blocking, Step의 직접 Provider 호출, Flow 간 직접 구동, 대기 Timeout·취소, Durable Step 복구 정책 |
| 009~015 | `goTo` 대상, Event Callback 책임, Engine·Worker 수명주기 소유, 구독 방식, Step ID 중복, `ExecutionContext` 오용, Step 인스턴스 공유 |
| 016~019 | 반복 Scheduler 승인, Guard Side Effect, EventStep Deadline과 Durable Await 복구 |
| 006~008 | Action Registry·Policy 우회, Audit 누락, 승인 필요 Action의 직접 실행을 검사하는 선택형 규칙 |

## 구현 범위

- JavaParser 기반 Java 소스 분석과 보수적인 Text Fallback
- ServiceLoader 기반 검사 규칙 등록
- 파일·행 번호가 포함된 Plain Text와 SARIF 출력
- 프로젝트별 규칙 활성화 및 심각도 설정
- 기존 위반을 통제된 상태로 관리하는 Baseline 파일
- 의도적인 반복 Scheduler를 표시하는 SOURCE 보존 승인 Annotation
- CLI와 Maven·Gradle Plugin
- 실제 임시 호스트 프로젝트를 이용한 Maven Invoker와 Gradle TestKit 검증
- Flower 저장소 자체 소스를 다시 검사하는 Reactor `verify`

## 빌드 연결

```text
소스 작성
→ Maven verify 또는 Gradle check
→ flower-check Java 소스 검사
→ flower-testkit 및 일반 테스트
→ 위반이 없을 때 패키징·배포
```

Maven Plugin은 `verify`, Gradle Plugin은 `check`에 연결됩니다. SARIF 결과는 CI의 코드 검사 결과로 전달할 수 있고, 새로 도입하는 프로젝트는 Baseline을 이용해 기존 위반과 신규 위반을 분리할 수 있습니다.

## 적용 범위

`flower-check`는 Flower 사용 규칙에 한정된 소스 검사기입니다. 일반 Java Linter, 보안 취약점 검사기, Runtime 권한 제어 또는 테스트 대체 도구가 아닙니다. 업무 정책과 실제 Side Effect의 정확성은 애플리케이션 테스트 및 [Flower Action Runtime]({{ '/projects/flower-action-runtime/' | relative_url }}) 같은 실행 통제 계층에서 별도로 검증합니다.

프로젝트 정보는 [Flower 오픈소스]({{ '/projects/flower/' | relative_url }})와 [Flower Agent Skills]({{ '/projects/flower-agent-skills/' | relative_url }}) 페이지에 정리했습니다.
