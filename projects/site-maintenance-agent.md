---
title: 사이트 유지보수 에이전트 서버
description: Slack의 장애 조사 요청을 Java가 고정된 절차로 해석하고, 현장 로그·등록 DB 조회·소스·지식을 수집해 근거가 검증된 보고서로 회신하는 유지보수 오케스트레이터입니다.
permalink: /projects/site-maintenance-agent/
period: 2026 · 설계 및 개발
category: 현장 유지보수 자동화 · Java Agent Orchestrator
role: Java 서버 전체 설계·개발, Flower Flow와 Action 통제, 현장 연동 및 검증
stack: Java 21 · Spring Boot · Flower · Flower Action Runtime · Flower AI Harness · Bloom · flower-check
---

## 프로젝트 개요

사이트 유지보수 에이전트 서버는 현장 시스템의 정기 점검과 장애 조사를 실행하는 로컬 Java 서버입니다. 단순히 Slack 메시지를 AI에 전달하는 챗봇이 아니라, **조사 절차와 실행 권한은 Java가 소유하고 AI는 준비된 증거를 분석하는 역할만 담당**하도록 구성했습니다.

| 항목 | 내용 |
|---|---|
| 자동 운영 | 사이트별 주기에 맞춰 VPN 연결, 로그 증분 수집, 분석, 보고서 생성과 통보 수행 |
| 대화형 조사 | Slack 요청의 대상·시간을 확인하고 로그·지식·소스·등록 DB 증거를 준비해 같은 Thread에 결과 회신 |
| 실행 통제 | 모든 외부 작업을 등록 Action으로 제한하고 정책·승인·중복 방지·감사 절차 적용 |
| 복구 | Flow 위치와 Action 실행 상태를 각각 영속화하고 서버 재기동 후 미완료 작업 재개 |
| 현장 검증 | ATCSS 현장 자료를 대상으로 로그·등록 DB 조회·소스 분석·보고서 생성까지 통합 경로 검증 |

<div class="project-metrics" aria-label="프로젝트 코드 구성">
  <div><strong>7</strong><span>Durable Flow</span></div>
  <div><strong>5</strong><span>Worker Lane</span></div>
  <div><strong>93</strong><span>Action Executor</span></div>
  <div><strong>94</strong><span>Java Test File</span></div>
</div>

<p class="scope-note">수치는 2026년 7월 분석 시점의 Java 소스 기준입니다. 현장 접속 정보, 계정, 원문 로그와 내부 보고서는 공개하지 않습니다.</p>

## 전체 아키텍처

<div class="architecture-diagram" role="img" aria-label="사이트 유지보수 에이전트 서버 전체 아키텍처">
  <div class="architecture-layer architecture-entry">
    <p class="diagram-label">요청과 운영</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>Slack</strong><span>@agent · Thread · 승인 버튼</span></div>
      <div><strong>Local Web UI</strong><span>타깃 · 실행 · 승인 · 조사 상태</span></div>
      <div><strong>Scheduler</strong><span>사이트별 자동화 주기</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>검증된 요청</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">Java Application</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>Conversation</strong><span>허용 사용자 · 중복 제거 · 의도 결정</span></div>
      <div><strong>Investigation</strong><span>조사 범위 · 증거 · 결과 검증</span></div>
      <div><strong>Automation</strong><span>수집 주기 · 실행 상태 · 보고</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>고정된 Flow 제출</span></div>
  <div class="architecture-layer architecture-flower">
    <p class="diagram-label">Flower · 실행 흐름</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>Flow / Step</strong><span>순서 · 분기 · 대기 · 재시도 · 종료</span></div>
      <div><strong>Checkpoint / Recovery</strong><span>JDBC 위치 저장 · 재기동 복구</span></div>
    </div>
    <div class="worker-lanes">
      <span>collector</span><span>analyzer</span><span>controller</span><span>conversation</span><span>investigation</span>
    </div>
  </div>
  <div class="diagram-connector"><span>논리 Action 요청</span></div>
  <div class="architecture-layer architecture-action">
    <p class="diagram-label">Flower Action Runtime · 실행 통제</p>
    <div class="control-pipeline">
      <span>Registry</span><span>입력 검증</span><span>Policy</span><span>승인</span><span>중복 방지</span><span>실행 직전 검증</span><span>감사</span>
    </div>
  </div>
  <div class="diagram-connector"><span>허용된 Executor만 실행</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">현장 연동과 결과</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>Evidence</strong><span>VPN · 로그 다운로드 · 등록 DB 조회 · 소스</span></div>
      <div><strong>Agent</strong><span>격리 Workspace · 구조화 결과 · 재검증</span></div>
      <div><strong>Output</strong><span>HTML/JSON 보고서 · Slack Thread 회신</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>영속 상태와 불변 Artifact</span></div>
  <div class="architecture-layer architecture-store">
    <p class="diagram-label">저장 경계</p>
    <div class="architecture-items architecture-items-2">
      <div><strong>PostgreSQL</strong><span>업무 상태 · Checkpoint · ActionRun · 승인 · 감사</span></div>
      <div><strong>Artifact Storage</strong><span>원본 · 정제 증거 · 출처 정보 · 보고서</span></div>
    </div>
  </div>
</div>

## Slack 요청부터 결과 회신까지

<div class="process-grid" aria-label="Slack 장애 조사 처리 순서">
  <article>
    <span>01</span>
    <h3>요청 수신</h3>
    <p>Socket Mode로 Mention과 Thread 답변을 수신합니다. 공개 Inbound HTTP Endpoint는 열지 않습니다.</p>
  </article>
  <article>
    <span>02</span>
    <h3>먼저 기록</h3>
    <p>Workspace·사용자 허용 목록과 Event ID를 검사하고 Inbox를 DB에 Commit한 뒤 Slack ACK를 반환합니다.</p>
  </article>
  <article>
    <span>03</span>
    <h3>요청 구체화</h3>
    <p><code>conversation-turn</code> Flow가 대상, 시간 범위와 요청 목적을 구조화합니다. 필수 정보가 없으면 같은 Thread에서 다시 묻습니다.</p>
  </article>
  <article>
    <span>04</span>
    <h3>조사 생성</h3>
    <p>정보가 완성되면 조사 ID, 초기 권한 범위와 작업 공간을 만들고 <code>incident-investigation</code> Flow를 제출합니다.</p>
  </article>
  <article>
    <span>05</span>
    <h3>기준 증거 준비</h3>
    <p>Java가 사이트 설정에 따라 VPN, 요청 시간대 로그, 지식과 허용된 소스 범위를 준비합니다.</p>
  </article>
  <article>
    <span>06</span>
    <h3>분석 반복</h3>
    <p>Agent가 정제된 증거를 분석하고 부족한 자료는 논리 EvidenceRequest로 제안합니다. 최대 5개 Round 안에서 재수집·재분석합니다.</p>
  </article>
  <article>
    <span>07</span>
    <h3>결과 검증</h3>
    <p>근거 Artifact, Line 범위와 Hash를 서버가 다시 확인합니다. Raw SQL, 원격 변경과 접속 정보가 포함된 결과는 거부합니다.</p>
  </article>
  <article>
    <span>08</span>
    <h3>보고와 피드백</h3>
    <p>HTML·JSON 보고서를 저장하고 같은 Slack Thread에 요약을 보냅니다. 확인 또는 정정 피드백도 별도 영속 Flow로 처리합니다.</p>
  </article>
</div>

## Slack에서 시작하는 계층 구조

<div class="hierarchy-diagram" role="img" aria-label="Slack부터 실행 계층까지의 구조">
  <div class="hierarchy-node hierarchy-root">
    <span>사용자 접점</span>
    <strong>Slack @agent / Thread Reply</strong>
  </div>
  <div class="hierarchy-arrow">↓</div>
  <div class="hierarchy-node">
    <span>입력 경계</span>
    <strong>Slack Socket Mode Adapter</strong>
    <small>Allowlist · Deduplication · Record-before-ACK</small>
  </div>
  <div class="hierarchy-arrow">↓</div>
  <div class="hierarchy-node hierarchy-flower-node">
    <span>대화 Flow</span>
    <strong>conversation-turn</strong>
    <small>Prepare → Agent Understanding → Validate → Resolve → Reply</small>
  </div>
  <div class="hierarchy-branch">
    <div>
      <strong>정보 부족</strong>
      <span>현장·시간 범위를 질문하고 종료</span>
    </div>
    <div>
      <strong>정보 완성</strong>
      <span>조사 생성 후 다음 Flow 제출</span>
    </div>
  </div>
  <div class="hierarchy-arrow">↓</div>
  <div class="hierarchy-node hierarchy-flower-node">
    <span>조사 Flow</span>
    <strong>incident-investigation</strong>
  </div>
  <div class="hierarchy-steps">
    <span>계획</span><span>기준 증거</span><span>분석</span><span>추가 증거·승인</span><span>검증</span><span>보고</span>
  </div>
  <div class="hierarchy-arrow">↓</div>
  <div class="hierarchy-node hierarchy-action-node">
    <span>공통 실행 경계</span>
    <strong>Governed Action Service</strong>
    <small>모든 외부 작업은 Action Runtime Pipeline을 통과</small>
  </div>
</div>

## Flower가 담당하는 역할

Flower는 업무 판단이나 현장 접속 도구가 아니라 **장시간 실행되는 Java 업무의 진행 상태를 관리하는 실행 엔진**으로 사용했습니다.

| Flower가 담당 | 적용 내용 |
|---|---|
| 실행 구조 | 업무를 Flow와 이름이 있는 Step으로 고정하고 순서·분기·반복을 코드로 정의 |
| 비차단 대기 | Agent 완료, 승인 결정과 다음 실행 시각을 Worker Thread를 점유하지 않고 확인 |
| 영속 위치 | 현재 Step, 실행 문맥과 정의 버전을 JDBC Checkpoint에 저장 |
| 재기동 복구 | 7개 Flow Factory를 Registry에 등록하고 미완료 Flow를 같은 ID로 복원 |
| 실행 격리 | 수집·분석·제어·대화·조사를 5개 Worker Lane으로 분리 |
| 이벤트 전달 | Bloom Adapter로 승인·상태 변경 Event를 대기 중인 Step에 전달 |
| 구현 규칙 검사 | Maven `verify`에서 `flower-check`를 실행해 Blocking, 복구 정책과 우회 호출을 검사 |

Flower가 직접 담당하지 않는 영역도 분리했습니다.

- 현장·조사·승인 상태의 원본은 PostgreSQL 업무 테이블이 관리합니다.
- VPN, 로그 수집, DB 조회, Agent 실행과 Slack 전송은 Action Executor가 담당합니다.
- 실행 허용 여부와 승인은 Flower가 아니라 Flower Action Runtime의 Policy·Approval 경계가 결정합니다.
- AI는 Flow나 Step을 생성하지 않으며, 어떤 Executor를 호출할지도 결정하지 않습니다.

## 구현한 Flow

| Flow | 역할 |
|---|---|
| `site-automation` | 사이트별 다음 실행 시각을 기다리고 유지보수 Cycle을 반복 |
| `site-maintenance` | VPN, 로그 수집, 정리, 보존 처리와 분석 제출 |
| `log-analysis` | 정기 분석 Agent 실행, 결과 검증, 보정 분석과 보고서 발행 |
| `ops-maintenance` | 보존·복구·주간 요약 등 서버 자체 유지관리 |
| `conversation-turn` | Slack 요청 이해, 누락 정보 질문, 조사 생성과 Thread 답변 |
| `incident-investigation` | 계획, 기준 증거, 최대 5회 증거 보강, 검증, 보고와 연결 정리 |
| `knowledge-feedback-refinement` | 사용자의 정정 내용을 최대 2회 구조화하고 지식 후보 발행 절차로 연결 |

모든 주요 Step에는 `REENTER_IDEMPOTENT` 복구 정책을 지정했습니다. Flow 제출은 Outbox와 동일 Flow ID 확인을 사용해 서버 재기동이나 중복 요청 시 같은 업무가 다시 실행되지 않도록 구성했습니다.

## Action Runtime 통제 구조

Flow에서 실행되는 부수효과는 `FlowActionGateway`를 통해 하나의 공통 경계로 들어갑니다.

<ol class="flow-list">
  <li>논리 Action ID와 입력, 실행 주체, Target, Idempotency Key를 구성합니다.</li>
  <li>Registry에서 서버에 등록된 Action인지 확인합니다.</li>
  <li>입력 Schema와 Action Definition을 검증합니다.</li>
  <li>요청 출처, 사용자 권한, 사이트별 정책과 조사 Grant를 평가합니다.</li>
  <li>중복 실행이면 기존 ActionRun을 반환하고, 신규 실행이면 영속 Run을 생성합니다.</li>
  <li>범위 확대나 대량 수집은 승인 상태로 전환하고 Slack 또는 Web UI 결정을 기다립니다.</li>
  <li>승인 후 현재 정책과 업무 상태를 다시 검사한 뒤 Executor를 호출합니다.</li>
  <li>결과와 재시도 가능 여부, 감사·Trace를 저장합니다.</li>
</ol>

### 읽기 전용 현장 경계

| 구분 | 허용 | 차단 |
|---|---|---|
| 로그 | 등록 Source ID의 목록 조회와 다운로드 | Upload, Delete, Rename, 임의 Remote Path |
| DB | 사람이 등록한 Query ID와 형식이 정해진 Parameter | AI가 작성한 SQL, DML·DDL·PL/SQL |
| 소스 | 등록된 배포 버전과 허용 Subpath의 Snapshot | 코드 수정, Build, 배포 |
| Agent | 격리 Workspace의 `read`·`glob`·`grep`, 지정 결과 파일 작성 | Network, Shell, Credential, VPN·FTP·DB 직접 접근 |
| Slack | 원 요청 Thread의 정제된 질문·상태·결과 전송 | 원문 로그, Secret, 임의 Channel 전송 |

금지 기능은 승인을 받으면 실행되는 방식이 아니라 Action ID와 Executor 자체를 등록하지 않았습니다. 서버 기동 시 Action 계약을 검사해 허용하지 않은 DB·FTP 기능이나 중복 Action ID가 있으면 실행을 중단합니다.

## Agent 분석과 증거 검증

Agent에게는 현장에서 받은 원문 전체를 바로 전달하지 않습니다.

1. 원문 로그와 DB 결과는 변경하지 않는 Raw Artifact로 보관합니다.
2. Secret Redaction과 Line Mapping을 적용한 Observation Snapshot을 생성합니다.
3. 조사 시점의 사이트 지식과 배포 버전을 Retrieval Snapshot으로 고정합니다.
4. 허용된 Source Snapshot과 이전 Round의 증거 목록을 격리 Workspace에 배치합니다.
5. Flower AI Harness의 Agent CLI Provider를 통해 제한된 실행을 제출합니다.
6. 결과 JSON을 Schema와 업무 규칙으로 검증합니다.
7. 보고서에 사용된 Artifact ID, 파일, Line 범위와 Excerpt Hash를 Java가 다시 계산합니다.

Agent가 추가 자료가 필요하다고 판단하면 실제 경로나 SQL이 아니라 다음 세 종류 중 하나의 논리 요청만 반환합니다.

- `SERVER_LOG`: 등록된 Log Source에서 지정 시간 범위 추가 수집
- `REGISTERED_DB_QUERY`: 등록 Query ID와 검증 가능한 Parameter로 조회
- `SOURCE_SCOPE`: 등록된 소스 범위 Snapshot 추가

이 요청은 Java가 현재 조사 Grant와 Catalog에 대조한 뒤 실제 Action으로 변환합니다. Grant를 넘는 범위는 승인 대상으로 분리하고, 승인해도 허용할 수 없는 요청은 즉시 거부합니다.

## 자동 운영 모드

사이트별 `Automation On` 상태에서는 아래 Cycle이 지정 주기마다 실행됩니다.

<div class="compact-process" aria-label="자동 운영 주기">
  <span>실행 시각 대기</span><b>→</b><span>VPN 연결 또는 재사용</span><b>→</b><span>로그 증분 수집</span><b>→</b><span>정리·보존 처리</span><b>→</b><span>Agent 분석</span><b>→</b><span>보고서·통보</span>
</div>

- 원격 파일의 Identity와 Cursor를 저장해 이미 처리한 범위를 다시 받지 않습니다.
- 수집·정리·분석·보고 단계를 별도 Flow와 Action으로 나눠 실패 지점을 확인할 수 있습니다.
- VPN 공유 자원은 Lease로 관리하며, 같은 Group의 타깃이 동시에 접속하지 않도록 직렬화합니다.
- 다음 실행 시각과 Cycle 상태는 업무 DB에 저장하고, Flower Checkpoint는 현재 실행 위치를 저장합니다.

## 직접 개발한 Java 영역

- Spring Boot 기반 로컬 오케스트레이터와 설정·기동·복구 구조
- 7개 Flower Flow Factory와 5개 Worker Lane 구성
- Flow 제출 Outbox, 중복 방지와 재기동 복구
- 93개 Action Executor와 공통 Action Definition 계약
- Policy Gate, 승인, 실행 직전 검증, JDBC ActionRun과 감사 저장
- Slack Socket Mode Inbound, Record-before-ACK와 같은 Thread Outbound
- VPN Lease, 증분 로그 수집, 정리·보존, 등록 DB Query 실행
- 조사 Grant, EvidenceRequest 검증과 증거 Dispatch
- Agent 실행 Queue, 실행 한도, Workspace와 Structured Output 검증
- Knowledge Retrieval Snapshot과 정정 피드백·발행 승인 Flow
- Incident HTML/JSON 보고서와 진행 상태 UI
- Flower Testkit 기반 Flow 테스트, 외부 연동 Fake와 통합 테스트

Slack은 요청·승인·결과 전달을 위한 접점으로 사용했습니다. 이 프로젝트에서 강조하는 구현 범위는 Slack 제품 자체나 별도 화면 기술이 아니라, 그 요청을 안전하게 처리하는 Java 오케스트레이션·통제·증거 검증 계층입니다.

## 현장 검증 범위

ATCSS 현장 유지보수 데이터를 기준으로 다음 통합 경로를 실행해 결과 Artifact를 확인했습니다.

- Slack 요청에서 조사 ID와 별도 작업 공간 생성
- 지정 시간대 서버 로그 수집 및 증분·정제 처리
- 등록된 현장 DB Query 결과를 증거 Artifact로 추가
- 배포 소스 Snapshot과 계층형 현장 지식 제공
- Agent 분석 결과의 Schema, 인용 Line과 Hash 검증
- Incident HTML·JSON 보고서 생성과 Slack Thread 회신
- 실패·재시도·중복 Event·승인 대기·서버 재기동 복구 시나리오

검증 범위는 유지보수 조사와 보고까지입니다. 현장 코드 자동 수정, 원격 파일 변경, DB 변경과 자동 배포는 이 서버의 기능 범위에 포함하지 않았습니다.

## 관련 프로젝트

- [Flower 오픈소스]({{ '/projects/flower/' | relative_url }}) — Flow·Step, Worker, Checkpoint와 `flower-check`
- [Flower Action Runtime]({{ '/projects/flower-action-runtime/' | relative_url }}) — Action 정책·승인·영속 실행과 감사
- [Flower AI Harness]({{ '/projects/flower-ai-harness/' | relative_url }}) — 제한된 Agent 실행과 Provider 수명주기
- [Bloom Event Bus]({{ '/projects/bloom/' | relative_url }}) — JVM 내부 Event 전달
- [Flower Agent Skills]({{ '/projects/flower-agent-skills/' | relative_url }}) — Flower 애플리케이션과 Action Runtime 적용 가이드

<p class="scope-note">이 페이지는 실제 저장소의 Java 소스, Flow 정의, Action 계약, 테스트와 현장 실행 Artifact를 기준으로 작성했습니다. 공개 페이지에는 구조와 직접 개발한 범위만 기록하고 현장 원문 데이터와 접속 정보는 포함하지 않았습니다.</p>
