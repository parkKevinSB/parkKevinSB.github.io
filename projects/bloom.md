---
title: Bloom Event Bus
description: 객체와 모듈 사이의 JVM 내부 이벤트를 전달하기 위해 만든 경량 Pure Java Publish/Subscribe 런타임입니다.
permalink: /projects/bloom/
period: 2026 - 현재
category: Java Event Bus · 오픈소스
role: API·Dispatch 규칙 설계, Core·Spring·Flower Adapter 개발 및 테스트
stack: Java 8, Maven, Publish/Subscribe, Spring Framework, Flower
---

## 프로젝트 개요

Bloom은 하나의 JVM 안에서 객체와 모듈 사이의 이벤트를 전달하는 경량 Java Event Bus입니다. Workflow 실행은 Flower가 담당하고, Bloom은 서로 직접 참조하지 않는 구성요소 사이에 Runtime Event를 전달합니다.

| 항목 | 내용 |
|---|---|
| 공개 저장소 | [github.com/flowerjvm/bloom](https://github.com/flowerjvm/bloom) |
| 공개 릴리스 | 0.1.1 |
| 라이선스 | Apache License 2.0 |
| 호환성 | Java 8 |
| 배포 상태 | 현재 소스 빌드 및 로컬 설치 방식 |

## Core 기능

- Event Class 기반 Typed Publish/Subscribe
- Event 상속 관계를 확장하지 않는 Exact Type Dispatch
- 명시적인 `Subscription` 반환과 해제
- 여러 Listener 중 하나가 실패해도 나머지 호출을 계속하는 오류 분리
- Listener Error Handler
- Marker Interface나 공통 Base Class가 필요 없는 일반 Java Event
- Feature·Workflow·Test 단위로 생성 가능한 Runtime Scope

## 동기·비동기 전달

### LocalEventBus

- Event를 발행한 Thread에서 즉시 Listener 호출
- 호출 순서와 결과를 확인하기 쉬운 동기 실행
- 외부 Scheduler 없이 테스트 가능

### AsyncEventBus

- 기존 Event Bus를 감싸고 Executor에서 Publish 실행
- Executor 생성과 종료는 사용하는 애플리케이션이 담당
- Event Bus가 임의의 Thread Pool을 소유하지 않도록 수명주기 분리

## Subscriber 구조

| 구성 | 역할 |
|---|---|
| `EventHandler<E>` | Event 하나를 처리하는 함수형 Handler |
| `AbstractTypedEventHandler<E>` | Event Type과 Subscription 수명주기를 Handler에 포함 |
| `AbstractEventSubscriber` | 한 객체가 보유한 여러 Subscription을 함께 관리 |
| `Subscription` | 등록 해제 Handle, 반복 Close 허용 |

## Spring 및 Flower 연동

### Spring

- `@EnableBloom` 설정
- `@Subscribe`가 적용된 Bean Method 검색
- 기존 EventBus Bean이 있으면 해당 Runtime 사용
- Spring Application Event로 변환하지 않고 Bloom Bus에 직접 등록

### Flower

- `bloom-flower-adapter`에서 Bloom EventBus를 Flower의 EventBus SPI로 연결
- 외부 Callback, 승인 결과와 Domain Event를 Flower의 대기 Step에 전달
- Bloom은 Event 전달만 담당하고 Flow 상태와 Timeout·Retry는 Flower가 관리

## 공개 모듈

| 모듈 | 역할 |
|---|---|
| `bloom-core` | 의존성 없는 Event Bus API와 구현 |
| `bloom-spring` | Spring Framework Annotation 연동 |
| `bloom-flower-adapter` | Flower EventBus SPI Adapter |

## 적용 범위와 제한

Bloom은 JVM 내부 Notification 전달용입니다. 분산 메시지 Broker, 영속 Queue, Retry Engine 또는 Transaction Manager가 아닙니다. 프로세스 재기동 후에도 보존되어야 하거나 다른 서비스로 전달해야 하는 Event는 Kafka 등 별도 메시징 시스템을 사용해야 합니다.

## 관련 페이지

- [Flower 오픈소스]({{ '/projects/flower/' | relative_url }})
- [Flower AI Harness]({{ '/projects/flower-ai-harness/' | relative_url }})
- [Flower JVM 프로젝트 역할 구분]({{ '/notes/flower-jvm-project-boundaries/' | relative_url }})

