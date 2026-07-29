---
title: 항만 현장 유지보수 AI 에이전트 서버
description: Slack에서 접수한 항만 현장 장애 조사 요청을 정해진 Java 실행 흐름으로 처리하고, 현장 자료를 수집·분석해 결과를 회신하는 Flower 기반 유지보수 서버입니다.
permalink: /projects/site-maintenance-agent/
period: 2026 · 설계 및 개발
category: 항만 현장 유지보수 자동화 · Java AI Orchestrator
role: Java 서버와 유지보수 Flow 설계·개발, Flower 기반 실행 관리, 현장 연동 및 검증
stack: Java 21 · Spring Boot · Flower · Flower Action Runtime · Flower AI Harness · Bloom · flower-check
---

## 프로젝트 개요

항만 운영 시스템의 정기 점검과 장애 조사를 지원하기 위해 구축한 로컬 Java 서버입니다. Slack에서 조사 요청을 받거나 정해진 일정에 따라 작업을 시작하고, 현장 로그·등록된 DB 조회 결과·배포 소스와 운영 지식을 모아 AI 분석 결과와 보고서를 제공합니다.

AI가 임의로 현장 시스템을 조작하는 구조가 아닙니다. Java 서버가 조사 순서와 실행 범위를 관리하고, AI는 서버가 준비한 자료를 분석하는 역할을 담당합니다.

| 항목 | 내용 |
|---|---|
| 요청 | Slack 대화 또는 등록된 일정으로 점검·조사 시작 |
| 자료 수집 | 현장 로그, 등록 DB 조회 결과, 배포 소스와 운영 지식 준비 |
| 실행 관리 | Flower Flow로 처리 순서, 대기와 재기동 복구 관리 |
| AI 분석 | 준비된 자료를 제한된 범위에서 분석하고 정해진 형식으로 결과 반환 |
| 결과 | 조사 결과와 보고서를 Slack Thread 및 운영 화면에 제공 |
| 현장 검증 | ATCSS 유지보수 자료를 이용해 수집부터 분석·보고까지 통합 흐름 검증 |

## 전체 구조

<div class="architecture-diagram" role="img" aria-label="항만 현장 유지보수 AI 에이전트 서버 구조">
  <div class="architecture-layer architecture-entry">
    <p class="diagram-label">요청</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>Slack</strong><span>장애 조사 요청 · 추가 질문 · 결과 확인</span></div>
      <div><strong>Schedule</strong><span>등록된 현장 점검 주기</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>요청 확인</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">Java 유지보수 서버</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>조사 관리</strong><span>대상 · 시간 범위 · 진행 상태 관리</span></div>
      <div><strong>실행 통제</strong><span>허용된 작업과 승인 범위 확인</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>정해진 실행 흐름</span></div>
  <div class="architecture-layer architecture-flower">
    <p class="diagram-label">Flower</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>Flow / Step</strong><span>수집 · 분석 · 검증 · 보고 순서 관리</span></div>
      <div><strong>Checkpoint / Recovery</strong><span>진행 위치 저장 · 재기동 후 재개</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>통제된 수집과 분석</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">현장 자료와 AI 분석</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>Evidence</strong><span>로그 · 등록 DB 조회 · 소스 · 운영 지식</span></div>
      <div><strong>AI Agent</strong><span>자료 분석 · 결과 구조화</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>검증된 결과</span></div>
  <div class="architecture-layer architecture-store">
    <p class="diagram-label">결과</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>Slack Thread</strong><span>진행 상태 · 조사 결과 회신</span></div>
      <div><strong>Report</strong><span>근거가 포함된 장애 조사 보고서</span></div>
    </div>
  </div>
</div>

## 처리 흐름

<div class="process-grid" aria-label="항만 현장 장애 조사 처리 순서">
  <article>
    <span>01</span>
    <h3>요청 접수</h3>
    <p>Slack 요청 또는 등록된 일정으로 유지보수 작업을 시작합니다.</p>
  </article>
  <article>
    <span>02</span>
    <h3>조사 범위 확인</h3>
    <p>대상 현장과 시간 범위를 확인하고 정보가 부족하면 추가 질문을 보냅니다.</p>
  </article>
  <article>
    <span>03</span>
    <h3>현장 자료 준비</h3>
    <p>허용된 범위에서 로그, 등록 DB 조회 결과, 소스와 운영 지식을 준비합니다.</p>
  </article>
  <article>
    <span>04</span>
    <h3>AI 분석</h3>
    <p>준비된 자료를 분석하고 원인 후보와 확인 근거를 정해진 형식으로 반환합니다.</p>
  </article>
  <article>
    <span>05</span>
    <h3>결과 검증</h3>
    <p>분석 결과가 실제 수집 자료를 근거로 작성됐는지 Java 서버에서 확인합니다.</p>
  </article>
  <article>
    <span>06</span>
    <h3>보고</h3>
    <p>조사 결과를 보고서로 저장하고 요청이 시작된 Slack Thread에 회신합니다.</p>
  </article>
</div>

## Flower의 역할

Flower는 AI 모델이나 현장 접속 도구가 아니라, 장시간 유지보수 작업의 실행 순서와 상태를 관리하는 Java Flow Runtime으로 사용했습니다.

| 역할 | 적용 내용 |
|---|---|
| 실행 순서 | 자료 수집, 분석, 결과 검증과 보고 단계를 Flow와 Step으로 관리 |
| 대기 처리 | 외부 작업이나 사용자 응답을 기다리는 동안 Worker를 점유하지 않도록 처리 |
| 중단 복구 | 현재 진행 위치를 저장하고 서버 재기동 후 미완료 작업 재개 |
| 작업 분리 | 수집·분석·대화 등 성격이 다른 작업의 실행 영역 분리 |
| 규칙 검사 | `flower-check`로 Flow 사용 규칙과 복구 정책을 빌드 시점에 검사 |

외부 작업은 Flower Action Runtime을 통해 허용 범위와 승인 여부를 확인하고, AI 실행은 Flower AI Harness를 통해 제출·결과 검증 과정을 관리합니다. Bloom은 작업 상태 변경과 대기 중인 Flow 사이의 이벤트 전달에 사용했습니다.

## 주요 개발 범위

- Spring Boot 기반 유지보수 오케스트레이션 서버
- Flower Flow를 이용한 작업 실행과 재기동 복구
- Slack 요청·추가 질문·진행 상태·결과 회신
- 현장 로그, 등록 DB 조회 결과와 소스 자료 수집
- 실행 정책·승인과 읽기 전용 현장 접근 통제
- AI 분석 결과 검증과 장애 조사 보고서 생성

## 현장 검증

ATCSS 유지보수 자료를 이용해 Slack 요청, 현장 자료 수집, AI 분석, 결과 검증과 보고서 회신까지 전체 흐름을 검증했습니다. 작업 실패나 서버 재기동이 발생한 경우에도 저장된 진행 위치에서 조사를 이어갈 수 있도록 확인했습니다.

검증 범위는 장애 조사와 보고까지입니다. 현장 코드 자동 수정, 원격 파일 변경, DB 변경과 자동 배포는 기능 범위에 포함하지 않았습니다.

## 관련 프로젝트

- [Flower 오픈소스]({{ '/projects/flower/' | relative_url }}) — 작업 실행 순서와 재기동 복구
- [Flower Action Runtime]({{ '/projects/flower-action-runtime/' | relative_url }}) — 실행 정책·승인·감사
- [Flower AI Harness]({{ '/projects/flower-ai-harness/' | relative_url }}) — Agent 실행과 결과 검증
- [Bloom Event Bus]({{ '/projects/bloom/' | relative_url }}) — JVM 내부 Event 전달
- [Flower Agent Skills]({{ '/projects/flower-agent-skills/' | relative_url }}) — Flower 프로젝트 적용 가이드

<p class="scope-note">공개 페이지에는 프로젝트의 목적, 전체 흐름과 담당 범위만 정리했으며 현장 원문 데이터, 접속 정보와 내부 구현 세부사항은 포함하지 않았습니다.</p>
