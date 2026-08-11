---
title: 항만 현장 유지보수 AI 에이전트 서버
description: Slack에서 접수한 항만 현장 장애 조사 요청을 정해진 Java 실행 흐름으로 처리하고, 현장 자료를 수집·분석해 결과를 회신하는 Flower 기반 유지보수 서버입니다.
permalink: /projects/site-maintenance-agent/
period: 2026 · 설계 및 개발
category: 항만 현장 유지보수 자동화 · Java AI Orchestrator
role: Java 서버와 유지보수 Flow·AI Agent 실행 계층 설계·개발, 현장 연동 및 검증
stack: Java 21 · Spring Boot · Flower · Flower Action Runtime · Flower AI Harness · Codex SDK · Claude Agent SDK
---

## 프로젝트 개요

항만 운영 시스템의 정기 점검과 장애 조사를 지원하기 위해 구축한 로컬 Java 서버입니다. Slack에서 조사 요청을 받거나 정해진 일정에 따라 작업을 시작하고, 현장 로그·등록된 DB 조회 결과·배포 소스와 운영 지식을 모아 AI 분석 결과와 보고서를 제공합니다.

별도 Agent Runner에서 Codex SDK와 Claude Agent SDK를 실행하고, Flower AI Harness가 Agent 작업의 제출·상태 확인·결과 검증을 관리하도록 구성했습니다. 외부 작업은 Flower Action Runtime을 통해 허용 범위와 승인 여부를 통제합니다.

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

## 대화형 장애 조사 흐름

Slack에서 시작한 장애 조사는 대화 처리와 실제 조사를 별도 Flow로 실행합니다. 대상 현장, 조사 시간과 요청 목적이 확인되기 전에는 VPN이나 현장 자료 수집을 시작하지 않습니다.

<div class="incident-flow" aria-label="Slack 대화부터 현장 자료 수집, AI 분석, 보고까지의 장애 조사 흐름">
  <section class="incident-phase">
    <header>
      <span>1</span>
      <div>
        <strong>요청 접수와 정보 확인</strong>
        <small>Slack Conversation Flow</small>
      </div>
    </header>
    <div class="incident-steps incident-steps-4">
      <article>
        <b>01</b>
        <h3>대화 시작</h3>
        <p>Slack 요청을 중복 없이 저장한 뒤 짧게 응답하고, 별도의 대화 처리 Flow를 시작합니다.</p>
      </article>
      <article>
        <b>02</b>
        <h3>AI 요청 이해</h3>
        <p>대화 이해 Agent가 최근 대화와 현재 메시지에서 대상 현장, 시간 범위와 조사 목적을 구조화합니다.</p>
      </article>
      <article>
        <b>03</b>
        <h3>부족한 정보 질문</h3>
        <p>현장이나 시간이 빠졌으면 같은 Thread에서 질문하고, 답변이 오면 저장된 문맥과 합쳐 다시 확인합니다.</p>
      </article>
      <article>
        <b>04</b>
        <h3>조사 시작</h3>
        <p>필수 정보가 갖춰지면 조사 ID와 허용 범위를 저장하고 Incident Flow를 제출합니다.</p>
      </article>
    </div>
    <div class="incident-loop-note">
      <span>정보 부족</span>
      <strong>추가 질문 → 사용자 답변 → 요청 내용 재확인</strong>
    </div>
  </section>

  <div class="incident-phase-connector"><span>대상 · 시간 · 목적 확정</span></div>

  <section class="incident-phase incident-phase-evidence">
    <header>
      <span>2</span>
      <div>
        <strong>현장 연결과 기준 증거 준비</strong>
        <small>Incident Baseline</small>
      </div>
    </header>
    <div class="incident-steps incident-steps-4">
      <article>
        <b>05</b>
        <h3>조사 계획 검증</h3>
        <p>계획 Agent의 결과를 Java 서버가 등록된 로그, 소스와 조회 항목에 대조하고 허용 범위 안으로 고정합니다.</p>
      </article>
      <article>
        <b>06</b>
        <h3>VPN 연결</h3>
        <p>Java 서버가 현장별 연결 설정과 공용 Lease를 확인한 뒤 등록된 VPN 연결을 실행합니다.</p>
      </article>
      <article>
        <b>07</b>
        <h3>기준 자료 수집</h3>
        <p>FTP 읽기 전용 수집으로 요청 시간대 로그를 확보하고, 배포 소스와 운영 지식 Snapshot을 준비합니다.</p>
      </article>
      <article>
        <b>08</b>
        <h3>분석 자료 구성</h3>
        <p>원본은 별도로 보존하고, 민감 정보 제거와 출처 연결이 끝난 자료만 분석 Workspace에 배치합니다.</p>
      </article>
    </div>
  </section>

  <div class="incident-phase-connector"><span>검증된 Evidence Workspace</span></div>

  <section class="incident-phase incident-phase-agent">
    <header>
      <span>3</span>
      <div>
        <strong>AI 분석과 추가 증거 반복</strong>
        <small>Java Orchestrator · AI Harness · Node Agent Runner</small>
      </div>
    </header>
    <div class="runner-pipeline" aria-label="Java 서버에서 Claude 또는 Codex SDK까지의 Agent 실행 계층">
      <span>Flower Step</span>
      <span>agent.run</span>
      <span>AI Harness</span>
      <span>Node Agent Runner</span>
      <span>Claude Agent SDK<br>또는 Codex SDK</span>
    </div>
    <div class="incident-steps incident-steps-3">
      <article>
        <b>09</b>
        <h3>분석 실행</h3>
        <p>Java 서버가 실행 요청을 제한된 Queue에 등록하고, 분석 회차마다 Node Runner 프로세스 하나를 시작합니다.</p>
      </article>
      <article>
        <b>10</b>
        <h3>격리된 자료 분석</h3>
        <p>설정에 따라 Claude 또는 Codex가 실행됩니다. Agent는 네트워크 없이 읽기 전용 Workspace만 확인합니다.</p>
      </article>
      <article>
        <b>11</b>
        <h3>결과 검증</h3>
        <p>Java 서버가 결과 형식과 인용된 파일·구간을 실제 Evidence와 대조해 근거가 없는 결론을 차단합니다.</p>
      </article>
    </div>
    <div class="evidence-loop" aria-label="AI 추가 증거 요청 처리 순서">
      <div>
        <span>추가 확인 필요</span>
        <strong>AI Evidence Request</strong>
        <small>필요한 로그·등록 조회·소스 범위만 제안</small>
      </div>
      <i>→</i>
      <div>
        <span>Java 검증</span>
        <strong>범위 · 권한 · 한도 확인</strong>
        <small>허용 범위 초과 시 승인 대기</small>
      </div>
      <i>→</i>
      <div>
        <span>등록 Action 실행</span>
        <strong>FTP · DB 조회 · Source</strong>
        <small>새 Evidence를 Workspace에 추가</small>
      </div>
      <i>↺</i>
      <div>
        <span>다음 회차</span>
        <strong>새 Agent Run</strong>
        <small>추가 자료를 포함해 다시 분석</small>
      </div>
    </div>
    <p class="incident-boundary-note">AI는 VPN 명령, FTP 경로, DB 접속 정보나 SQL을 직접 실행하지 않습니다. 필요한 자료를 논리적인 요청으로 반환하고, 실제 외부 작업은 Java 서버가 등록된 Action으로 처리합니다.</p>
  </section>

  <div class="incident-phase-connector"><span>근거 검증 완료</span></div>

  <section class="incident-phase incident-phase-finish">
    <header>
      <span>4</span>
      <div>
        <strong>보고와 연결 정리</strong>
        <small>Report · Cleanup · Slack Reply</small>
      </div>
    </header>
    <div class="incident-steps incident-steps-3">
      <article>
        <b>12</b>
        <h3>보고서 생성</h3>
        <p>검증된 결과와 근거를 이용해 Java 서버가 HTML·JSON 조사 보고서를 생성합니다.</p>
      </article>
      <article>
        <b>13</b>
        <h3>VPN 연결 해제</h3>
        <p>조사가 사용한 Lease를 반납하고 필요한 경우 VPN을 해제합니다. 실패나 취소 시에도 정리 경로를 실행합니다.</p>
      </article>
      <article>
        <b>14</b>
        <h3>결과 회신</h3>
        <p>요청이 시작된 Slack Thread에 결과 요약과 보고서 정보를 보내고 실행 이력과 분석 기록을 보관합니다.</p>
      </article>
    </div>
  </section>
</div>

정기 점검은 Slack 대화 대신 등록된 일정에서 시작하지만, `VPN 연결 → 로그 증분 수집 → 분석 자료 구성 → Agent 분석 → 보고 → 연결 정리`의 동일한 실행 경계를 사용합니다.

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
