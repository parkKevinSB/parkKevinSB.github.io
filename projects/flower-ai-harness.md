---
title: Flower AI Harness
description: AI 모델이나 외부 에이전트가 수행하는 하나의 업무를 Flower Flow로 구성하고 검증·재시도·취소·복구할 수 있게 만든 Java 프레임워크입니다.
permalink: /projects/flower-ai-harness/
period: 2026 - 현재
category: Java AI 실행 프레임워크 · 오픈소스
role: 프로젝트 설계·개발, 테스트, 문서화 및 Maven Central 배포
stack: Java 21, Maven, Flower, Spring AI, Jackson, OpenAI Java, Anthropic Java
---

## 프로젝트 개요

Flower AI Harness는 AI 모델 호출이나 외부 에이전트 실행 하나를 통제 가능한 Flower 작업 흐름으로 구성하는 Java 프레임워크입니다. 모델을 직접 구현하지 않고, Provider 호출 전후의 실행 수명주기를 담당합니다.

| 항목 | 내용 |
|---|---|
| 공개 저장소 | [github.com/flowerjvm/flower-ai-harness](https://github.com/flowerjvm/flower-ai-harness) |
| 공개 버전 | 0.1.2 |
| 배포 | [Maven Central](https://central.sonatype.com/artifact/io.github.flowerjvm/flower-ai-harness-core/0.1.2) |
| 라이선스 | Apache License 2.0 |
| 기반 | Java 21, Flower 0.1.1 |

## 처리 흐름

<ol class="flow-list">
  <li>입력 데이터와 Prompt Version을 기준으로 요청을 준비합니다.</li>
  <li>Provider Gateway에 비동기로 작업을 제출합니다.</li>
  <li>Flower Worker Tick에서 완료 여부만 확인합니다.</li>
  <li>응답을 지정된 구조의 Java 객체로 변환하고 검증합니다.</li>
  <li>검증 결과와 시도 횟수에 따라 재시도, Prompt 보정 또는 모델 변경을 결정합니다.</li>
  <li>완료된 결과에서 Finding을 추출해 애플리케이션의 저장·발행 경계로 전달합니다.</li>
</ol>

## 직접 개발한 기능

### AI 실행 계약

- `AiHarnessSpec<I, T>` 기반 입력·출력 Type 정의
- Prompt Builder와 Prompt Version 관리
- 기본 Model ID와 Provider Option 설정
- `AiModelGateway`와 `AiModelCall` 기반 비동기 제출·조회
- 실행 Context와 Run Snapshot 제공

### 결과 검증과 재시도

- Jackson 기반 Structured Output 변환·검증
- 검증 실패 원인을 반영한 Prompt Refine
- 최대 시도 횟수와 Attempt Budget
- Provider 또는 Model Fallback
- 취소와 실행 시간 제한
- 동시 실행 수와 Resource 제한

### 결과 전달

- 도메인에 종속되지 않는 Finding 모델
- 결과에서 Finding을 추출하는 Extractor
- 애플리케이션이 구현하는 Finding Sink
- Trace와 실행 상태 조회
- 재기동 시 처리 정책과 Run Store 확장 지점

### 테스트

- 실제 Provider를 호출하지 않는 Fake Gateway
- 제출·조회·실패·재시도·취소 시나리오
- 결정론적인 Flower Tick 기반 테스트
- Sample 모듈의 End-to-end 실행 시나리오

## Provider 및 연동 모듈

| 모듈 | 역할 |
|---|---|
| `flower-ai-harness-core` | 실행 수명주기, Gateway, 정책, 상태와 Flow Factory |
| `flower-ai-harness-validator-jackson` | JSON Structured Output 검증 |
| `flower-ai-harness-test` | Fake Provider와 테스트 지원 |
| `flower-ai-harness-spring-ai` | Spring AI 연동 |
| `flower-ai-harness-provider-agent-cli` | Codex·Claude 등 외부 CLI Agent 실행 |
| `flower-ai-harness-provider-openai-compatible` | OpenAI 호환 Chat Completions API |
| `flower-ai-harness-provider-openai` | OpenAI 공식 Java SDK |
| `flower-ai-harness-provider-anthropic` | Anthropic 공식 Java SDK |
| `flower-ai-harness-spring-boot-starter` | Spring Boot 자동 설정 |

## 적용 범위와 제한

Flower AI Harness는 한 번의 AI 업무를 신뢰성 있게 실행하기 위한 프레임워크입니다. 자율 에이전트 플랫폼, RAG 시스템, 범용 Workflow Engine 또는 도메인 문서 라이브러리가 아닙니다.

Prompt, 업무 데이터, 영속 저장, UI와 최종 감사 기록은 사용하는 애플리케이션이 관리합니다. Provider의 장시간 호출은 별도 실행 영역에서 수행하고 Flower Worker는 제출과 상태 확인만 처리합니다.

## 관련 페이지

- [사이트 유지보수 에이전트 서버]({{ '/projects/site-maintenance-agent/' | relative_url }}) — 제한된 Agent 실행과 결과 검증을 실제 조사 Flow에 적용한 프로젝트
- [Flower 오픈소스]({{ '/projects/flower/' | relative_url }})
- [Flower Action Runtime]({{ '/projects/flower-action-runtime/' | relative_url }})
- [ArchDox]({{ '/projects/archdox/' | relative_url }})
- [Flower JVM 프로젝트 역할 구분]({{ '/notes/flower-jvm-project-boundaries/' | relative_url }})
