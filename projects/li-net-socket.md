---
title: LI Net Socket
description: Netty I/O와 업무 메시지 처리를 분리하고 세션별 Read/Write Queue를 외부 Task가 처리하도록 설계한 Java TCP Socket Runtime입니다.
permalink: /projects/li-net-socket/
period: 2026 · 설계 및 개발
category: 공용 TCP Socket Runtime · Agent Skill
role: 컨셉·구조·API 경계·검증 기준 설계, Agent Skill 작성, Agent-assisted 구현
stack: Java 8 · Netty · Spring Boot Starter · Gradle · JUnit
---

## 프로젝트 개요

기존 사내 소켓 라이브러리를 사용하면서 확인한 Thread 관리, 메시지 처리 결합과 확장 방식의 문제를 개선하기 위해 새로 설계한 Netty 기반 TCP Socket Runtime입니다.

핵심은 **네트워크 I/O와 업무 처리를 분리하는 것**입니다. Netty I/O Thread는 수신 데이터를 세션별 Read Queue에 넣는 역할까지만 담당합니다. 메시지 해석과 업무 처리는 애플리케이션의 별도 Task 또는 Worker가 Queue를 꺼내 실행하고, 송신 메시지도 세션별 Write Queue를 거쳐 비동기로 전송합니다.

## 핵심 처리 구조

<div class="compact-process" aria-label="LI Net Socket 메시지 처리 흐름">
  <span>Netty I/O<br>TCP Frame 수신</span><b>→</b><span>Session<br>Read Queue</span><b>→</b><span>외부 Task / Worker<br>Queue Drain</span><b>→</b><span>Message Route<br>Processor 실행</span><b>→</b><span>Session<br>Write Queue</span><b>→</b><span>Netty I/O<br>비동기 송신</span>
</div>

| 구성 | 역할 |
|---|---|
| Endpoint | TCP Server·Client 연결과 시작·종료 관리 |
| Session | 연결별 상태와 제한된 Read/Write Queue 관리 |
| Runtime | Session·Endpoint·Protocol과 처리 경계 분리 |
| Protocol Adapter | 수신 Frame 변환과 송신 메시지 Encoding |
| Message Route | 메시지 종류와 Decoder·Processor 연결 |
| Task / Worker | Queue를 꺼내 메시지 처리와 송신을 실행 |

하나의 Session에 Read Queue와 Write Queue를 각각 두어 연결별 메시지 순서를 유지합니다. Queue 용량을 제한하고 처리 속도가 수신 속도를 따라가지 못하면 읽기를 조절하도록 구성해, 메시지가 계속 누적되는 상황도 제어할 수 있게 했습니다.

## 기존 방식에서 개선한 점

| 기존 방식 | 개선한 구조 |
|---|---|
| 연결 객체가 자체 Thread에서 읽기·쓰기·업무 처리를 함께 수행 | Netty I/O와 외부 업무 처리 Thread 분리 |
| 수신 직후 Listener나 Handler를 직접 호출 | 세션별 Read Queue 적재 후 Task가 처리 |
| 연결 객체 내부의 송신 목록과 반복 전송 Loop | 세션별 Write Queue와 비동기 전송 |
| 연결·세션·메시지 책임이 한 구조에 결합 | Runtime·Endpoint·Session·Protocol 책임 분리 |
| 메시지 분기와 변환 방식이 구현체마다 달라짐 | 명시적인 Route와 메시지별 Processor 사용 |
| 프로젝트마다 연결과 처리 코드를 다시 구성 | Spring Boot Starter, 실행 예제와 Agent Skill 제공 |

새 라이브러리는 TCP 통신 자체에만 책임을 두고, 업무 ACK·재시도·중복 방지와 영속화는 사용하는 애플리케이션이 결정하도록 경계를 분리했습니다.

## 메시지별 Processor

<div class="compact-process" aria-label="메시지 라우팅과 Processor 실행 흐름">
  <span>Protocol Adapter</span><b>→</b><span>Message Type 확인</span><b>→</b><span>DTO 변환</span><b>→</b><span>전용 Processor</span><b>→</b><span>응답 또는 상태 처리</span>
</div>

Protocol Adapter에는 메시지 종류별 업무 분기를 넣지 않습니다. Route에서 메시지 Key, Decoder와 Processor의 관계를 명시적으로 등록하고, 서로 다른 메시지는 각각의 Processor가 담당하도록 구성했습니다.

이 구조를 사용하면 새로운 장비 전문을 추가할 때 기존 소켓 연결 코드를 변경하지 않고 DTO·Decoder·Processor만 추가할 수 있습니다.

## Runtime과 실행 방식

- 하나의 통신 경계만 필요한 애플리케이션은 기본 Runtime을 사용합니다.
- 서로 다른 장비나 프로토콜은 Runtime을 나누어 Session과 메시지 처리 상태를 격리할 수 있습니다.
- Netty I/O 자원은 기본적으로 공유하고, 별도 격리가 필요한 통신만 전용 자원을 사용할 수 있습니다.
- Endpoint 시작, Queue 처리 주기와 재시도 정책은 사용하는 애플리케이션의 Task 또는 Worker가 소유합니다.
- Spring Boot 프로젝트에서는 Starter가 공통 Runtime 구성을 제공합니다.

## Agent Skill

라이브러리 적용 규칙을 `li-net-socket` Agent Skill로 함께 작성했습니다. Codex, Claude Code와 Gemini CLI에서 필요한 Skill만 설치하고 버전과 파일 무결성을 확인할 수 있도록 구성했습니다.

<div class="compact-process" aria-label="LI Net Socket Agent Skill 적용 흐름">
  <span>Skill 설치·검증</span><b>→</b><span>프로토콜과<br>Endpoint 정의</span><b>→</b><span>Adapter·Route·<br>Processor 생성</span><b>→</b><span>Task / Worker 연결</span><b>→</b><span>빌드·테스트와<br>Checklist 검토</span>
</div>

Skill에는 다음 기준을 포함했습니다.

- Netty I/O Thread에서는 Queue 적재만 수행
- Session별 Read/Write Queue와 처리 Thread 경계 유지
- Protocol Adapter를 변환과 Routing 위임 수준으로 제한
- 메시지 종류별 Processor 분리
- 연결 Lifecycle, Queue 용량과 전송 실패 처리 확인
- Spring Boot 기본 구성과 Runtime 분리 기준
- 실행 예제 선택과 테스트 Checklist

이를 통해 코딩 에이전트가 라이브러리 구조를 임의로 바꾸거나 업무 코드를 I/O Thread에 넣지 않도록 하고, 새로운 프로젝트에도 동일한 형태로 빠르게 적용하고 테스트할 수 있게 했습니다.

## 개발 방식

기존 라이브러리의 문제를 기준으로 전체 컨셉, Thread 경계, Session Queue 구조, Runtime 분리와 메시지 Processor 방식을 직접 설계했습니다. 구현 코드, 테스트와 문서 작성에는 코딩 에이전트를 활용했으며, 공개 API와 적용 규칙, 검증 기준과 최종 구조는 직접 정리하고 검토했습니다.

Runtime과 Spring Boot Starter의 단위 테스트를 실행해 Session Queue, 메시지 Routing, 연결 Lifecycle, Runtime 격리와 자동 구성 경로를 확인했습니다.

<p class="scope-note">공개 페이지에는 라이브러리의 일반적인 설계 원칙과 직접 담당한 범위만 정리했으며, 사내 레거시 코드와 현장별 프로토콜은 포함하지 않았습니다.</p>
