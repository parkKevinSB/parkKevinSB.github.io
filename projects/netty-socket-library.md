---
title: Netty 기반 소켓 라이브러리
description: 네트워크 I/O와 업무 메시지 처리를 분리하고, 세션별 Queue를 애플리케이션 Task가 처리하도록 설계한 Java TCP 소켓 라이브러리입니다.
permalink: /projects/netty-socket-library/
period: 2026 · 설계 및 개발
category: Java TCP 통신 라이브러리
role: 컨셉·구조·적용 규칙 설계, Agent Skill 작성 및 구현 검토
stack: Java 8 · Netty · Spring Boot Starter · Gradle · JUnit
---

## 프로젝트 개요

기존 소켓 라이브러리에서 네트워크 처리와 업무 로직이 하나의 Thread에 결합되어 있던 구조를 개선하기 위해 만든 Netty 기반 Java TCP 라이브러리입니다.

Netty는 연결과 데이터 송수신을 담당하고, 실제 메시지 해석과 업무 처리는 애플리케이션의 별도 Task가 담당합니다. 두 영역 사이에는 연결별 Queue를 두어 네트워크 처리 속도와 업무 처리 속도가 서로 직접 영향을 주지 않도록 구성했습니다.

## 처리 구조

<div class="compact-process" aria-label="Netty 기반 소켓 라이브러리 처리 흐름">
  <span>TCP 연결과<br>데이터 수신</span><b>→</b><span>세션별<br>메시지 Queue</span><b>→</b><span>애플리케이션<br>Task</span><b>→</b><span>메시지별<br>Processor</span><b>→</b><span>응답<br>비동기 송신</span>
</div>

| 구성 | 역할 |
|---|---|
| 연결 관리 | TCP Server·Client 연결, 종료와 재연결 처리 |
| 세션 Queue | 연결별 수신·송신 메시지 보관과 처리 순서 유지 |
| 외부 Task | Queue에서 메시지를 가져와 업무 로직 실행 |
| Message Processor | 메시지 종류별 변환과 처리 로직 분리 |
| Spring Boot Starter | 애플리케이션에서 공통 구성을 바로 사용할 수 있도록 제공 |
| Agent Skill | 신규 프로젝트 적용과 테스트 기준 제공 |

## 기존 구조에서 개선한 점

| 기존 방식 | 개선한 방식 |
|---|---|
| 연결 Thread가 송수신과 업무 처리를 함께 수행 | Netty I/O와 애플리케이션 업무 처리 분리 |
| 수신 즉시 업무 Handler 호출 | 세션별 Queue를 거쳐 별도 Task에서 처리 |
| 연결 코드와 메시지 분기가 프로젝트마다 달라짐 | 공통 연결 구조와 메시지별 Processor 사용 |
| 새 프로젝트마다 통신 기반 코드를 다시 작성 | 공용 라이브러리, Spring Boot Starter와 적용 가이드 제공 |

Queue에는 처리 가능한 용량을 설정해 메시지가 제한 없이 쌓이지 않도록 했습니다. 새로운 장비나 프로토콜을 연결할 때는 공통 소켓 코드를 변경하지 않고 메시지 변환과 Processor를 추가하는 방식으로 확장할 수 있습니다.

## Agent Skill

코딩 에이전트가 라이브러리 구조와 Thread 경계를 유지하면서 신규 프로젝트에 적용할 수 있도록 Agent Skill을 함께 작성했습니다.

- 세션별 Queue와 외부 Task 연결
- 메시지 종류별 Processor 구성
- Spring Boot 기본 설정 적용
- 연결 종료, 재접속과 전송 실패 처리 확인
- 예제와 테스트 기준에 따른 검증

## 개발 범위

기존 라이브러리에서 확인한 문제를 바탕으로 네트워크 I/O와 업무 처리의 분리, 세션별 Queue와 메시지별 Processor 구조를 설계했습니다. 구현 코드와 테스트·문서 작성에는 코딩 에이전트를 활용했고, 적용 규칙과 최종 구조를 검토했습니다.

<p class="scope-note">공개 페이지에는 라이브러리의 목적과 주요 구조만 정리했으며, 사내 레거시 코드와 현장별 통신 프로토콜은 포함하지 않았습니다.</p>
