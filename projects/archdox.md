---
title: ArchDox
description: 감리 문서·체크리스트 작성, AI 검토와 문장 보정, 문서 생성, 법령 지식 갱신 및 MCP 연결을 제공하는 건축사사무소용 Workflow Platform입니다.
permalink: /projects/archdox/
period: 2026.05 - 현재
category: 개인 개발 · 공개 저장소 · AWS 운영
role: 기획 및 전체 Architecture, Backend·Frontend, 문서·AI·MCP 실행 구조 개발
stack: Java 21, Spring Boot, PostgreSQL, React, TypeScript, Flower, Bloom, AWS Lightsail, Docker, MinIO/S3
---

## 프로젝트 개요

ArchDox는 건축사사무소의 감리 업무를 프로젝트·현장·리포트 단위로 관리하고, 입력한 감리 데이터와 사진을 검토해 DOCX·PDF 문서로 생성하는 Platform입니다. 감리 문서 작성과 체크리스트, AI 문장 보정, 법령 지식 갱신, 외부 AI Agent용 MCP Gateway를 하나의 서비스로 구성했습니다.

| 항목 | 내용 |
|---|---|
| 실제 서비스 | [archdox.co.kr](https://archdox.co.kr/) |
| 업무 Web | [app.archdox.co.kr](https://app.archdox.co.kr/) |
| 공개 저장소 | [github.com/parkKevinSB/archdox](https://github.com/parkKevinSB/archdox) |
| 개발 상태 | 개인 개발 및 AWS 운영 |
| 배포 환경 | AWS Lightsail, Docker, Caddy/Nginx, PostgreSQL, MinIO/S3 |
| 주요 기술 | Java 21, Spring Boot 3.5, React, TypeScript, Flower, Bloom |

## 현재 구현 범위

- 프로젝트, 현장, 감리 대상과 담당자 관리
- 공사 감리일지의 공종·공정·감리 항목 입력 및 사진 증거 연결
- 감리 체크리스트 작성과 체크리스트 DOCX 출력
- 리포트 단계 저장, 제출, 수정 이력과 문서 생성 Revision 관리
- 필수 항목·사진·체크리스트를 먼저 확인하는 규칙 기반 사전 검토
- 감리 문장의 AI 보정 제안과 선택한 결과의 리포트 반영
- 감리 리포트의 HTML·DOCX·PDF 생성, Artifact 저장과 전달
- 대화에서 현장과 리포트를 선택하고 허용된 작업을 실행하는 Worker Chat
- 법제처 국가법령정보 Open API 기반 법령 수집·변경 비교·업데이트 요약
- 외부 AI Agent가 검토와 법령 조회 기능을 호출하는 Cloud MCP Server
- 사용자 Web, 관리자 Web, Cloud API와 문서 실행 Agent의 AWS 운영
- 사용자 환경에서 업무를 처리하는 Local AI Agent와 Agent MCP Server 확장 기획

## 전체 구조

<div class="architecture-diagram" role="img" aria-label="ArchDox 사용자 Web과 MCP, Cloud API, Flower 실행 계층, AI와 문서 실행 Agent, 데이터와 문서 저장소의 전체 구조">
  <div class="architecture-layer architecture-entry">
    <p class="diagram-label">사용자와 외부 연결</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>사용자 Web</strong><span>프로젝트 · 현장 · 감리 리포트</span></div>
      <div><strong>Worker Chat</strong><span>대화형 감리 문서 작업</span></div>
      <div><strong>Cloud MCP Client</strong><span>Codex · Claude · 외부 Agent</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>인증 · 권한 · 요청 Context</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">Cloud API와 업무 데이터</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>감리 업무</strong><span>현장 · 리포트 · 체크리스트 · 사진</span></div>
      <div><strong>ArchDox Engine</strong><span>Context 정규화 · 검토 · 근거</span></div>
      <div><strong>Cloud MCP Server</strong><span>도구 · Scope · 사용량 통제</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>장기 작업과 외부 호출 조정</span></div>
  <div class="architecture-layer architecture-flower">
    <p class="diagram-label">Flower 실행 계층</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>Document Flow</strong><span>검토 · 생성 · 전달</span></div>
      <div><strong>AI Harness</strong><span>결과 검증 · 재시도 · 보정</span></div>
      <div><strong>Legal Sync</strong><span>법령 수집 · 비교 · 갱신</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>통제된 AI 판단과 문서 실행</span></div>
  <div class="architecture-layer">
    <p class="diagram-label">실행 모듈</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>ArchDox Worker</strong><span>Policy · 승인 · Action Trace</span></div>
      <div><strong>ArchDox AI Harness</strong><span>문서 검토 · 문장 보정 · 계획</span></div>
      <div><strong>ArchDox Agent</strong><span>문서 변환 · 파일 · Storage 작업</span></div>
    </div>
  </div>
  <div class="diagram-connector"><span>업무 상태와 생성 결과 저장</span></div>
  <div class="architecture-layer architecture-store">
    <p class="diagram-label">저장과 결과</p>
    <div class="architecture-items architecture-items-3">
      <div><strong>PostgreSQL</strong><span>업무 상태 · Revision · 감사 기록</span></div>
      <div><strong>MinIO / S3</strong><span>사진 · Template · Artifact</span></div>
      <div><strong>DOCX / PDF</strong><span>감리 문서와 체크리스트</span></div>
    </div>
  </div>
</div>

## 감리 문서 처리 흐름

<ol class="flow-list">
  <li><strong>업무 Context 구성</strong><br>프로젝트와 현장을 선택하고 공종·공정·감리 항목, 체크리스트, 사진과 특이사항을 리포트에 저장합니다.</li>
  <li><strong>규칙 기반 사전 확인</strong><br>필수 입력, 체크리스트 완료 여부, 권한, Revision과 사진 증거를 코드로 먼저 검사합니다.</li>
  <li><strong>AI 검토와 문장 보정</strong><br>AI Harness가 감리 문장의 표현과 일관성을 검토합니다. 보정 결과는 제안으로 반환하며 사용자가 선택한 내용만 리포트에 저장합니다.</li>
  <li><strong>문서 생성 요청</strong><br>제출된 Revision과 Template, 출력 형식을 기준으로 문서 Job을 만들고 Flower Flow를 시작합니다.</li>
  <li><strong>Agent 실행</strong><br>등록된 Local 또는 Cloud-managed ArchDox Agent가 사진과 Template을 준비해 HTML·DOCX·PDF를 생성합니다.</li>
  <li><strong>보관과 전달</strong><br>생성된 Artifact를 S3 호환 Storage에 저장하고 Web에서 진행 상태, 이력과 결과 파일을 확인합니다.</li>
</ol>

## 현재 구조와 Local AI Agent 확장 계획

| 구성 | 역할 |
|---|---|
| `archdox-ai-harness` | 문서 사전 검토, 감리 문장 보정, 대화 계획과 운영 진단 등 AI 판단·언어 작업 |
| `archdox-worker` | 사용자·API·AI가 요청한 Action의 권한, Policy, 승인과 Trace를 확인한 뒤 기존 Domain Service 호출 |
| 현재 `archdox-agent` | Local Office 또는 Cloud 환경에서 문서 변환, 사진 수집, Artifact 전송과 Storage 접근을 수행하는 실행 Server |
| Local AI Agent 계획 | 사용자 환경의 업무 Context와 허용된 Tool을 이용해 감리 문서 작성·검토·생성 작업을 대화형으로 처리 |
| Agent MCP Server 계획 | Local AI Agent가 보유한 문서·파일·사무소 업무 기능을 MCP Tool로 제공 |
| Cloud MCP Server | Cloud의 ArchDox Engine, 법령 지식과 문서 검토 기능을 외부 AI Agent에 제공 |

현재 `archdox-agent`는 문서·파일 실행 기반을 담당합니다. 다음 단계에서는 이 Runtime에 AI 작업 계층과 Agent 전용 MCP Server를 결합해 사용자가 직접 운용하는 Local AI Agent로 확장할 계획입니다. Cloud MCP Server는 Local Agent의 MCP와 분리해 별도로 운영합니다.

## Cloud MCP Server

Cloud MCP Server는 ArchDox의 업무 DB를 임의로 조작하는 통로가 아니라, 인증된 외부 AI Agent가 정해진 검토 기능을 호출하는 Interface로 구현했습니다.

- 검토 Session 생성과 문서·Context 제출
- 입력 Context 정규화와 누락·모호한 항목 확인
- 감리 문서 검토와 구조화된 Finding 반환
- 동기화된 법령 데이터 검색과 조문 조회
- 공개된 법령 변경 요약 조회
- API Key Scope, 요청량 제한과 사용 이력 기록

Cloud MCP에서 받은 요청도 내부 Web과 동일한 Engine 검토 경계를 사용합니다. 외부 Agent가 내부 Entity나 Domain Action을 직접 실행하지 않도록 Context 정규화, Policy와 사용량 통제를 분리했습니다.

## Local AI Agent와 Agent MCP 계획

Local AI Agent는 Cloud MCP Server의 복제본이 아니라 사용자 환경에서 실제 업무를 처리하는 실행 주체로 기획했습니다.

<div class="process-grid">
  <article>
    <span>01</span>
    <h3>사용자 요청</h3>
    <p>감리 문서 작성, 체크리스트 생성, 기존 문서 검토와 수정 요청을 대화로 접수합니다.</p>
  </article>
  <article>
    <span>02</span>
    <h3>Local Context 구성</h3>
    <p>사용자가 허용한 프로젝트·현장·문서·사진과 Local Storage 자료를 업무 Context로 구성합니다.</p>
  </article>
  <article>
    <span>03</span>
    <h3>Agent MCP Tool 실행</h3>
    <p>문서 조회·작성·변환과 사무소별 업무 기능을 Agent 내부 MCP Tool로 제공하고 허용 범위에서 실행합니다.</p>
  </article>
  <article>
    <span>04</span>
    <h3>Cloud MCP 연계</h3>
    <p>법령 지식, 표준 검토와 Cloud Engine이 필요한 작업만 별도 Cloud MCP Server에 요청합니다.</p>
  </article>
</div>

Local AI Agent와 Cloud MCP Server의 호출 범위를 구분하고, 각각의 인증·권한·사용 이력을 분리하는 구조로 기획했습니다.

## 법령 지식 갱신

법령 정보는 별도의 Legal Domain으로 관리합니다.

1. 등록된 일정 또는 관리자 요청으로 법령 동기화 Flow 시작
2. 국가법령정보 Open API에서 대상 법령과 조문 수집
3. 법령 Version과 조문을 정규화하고 Hash 비교
4. 변경 조문과 변경 Set 저장
5. 감리 업무에 필요한 변경 요약과 관련 기준 생성
6. Web과 MCP의 법령 검색·변경 조회 기능에 제공

법령 정보는 감리 검토의 근거 Context로 사용하며, AI의 최종 법률 판단으로 취급하지 않습니다.

## Flower와 Bloom 적용

- Flower는 문서 검토·생성·전달, AI Harness, 운영 점검과 법령 동기화의 처리 순서와 대기 상태를 관리합니다.
- HTTP·모델 API·문서 변환처럼 오래 걸리는 작업은 별도 실행 영역에 제출하고 Flower Step은 완료 상태와 Timeout을 확인합니다.
- 업무 상태와 문서 Job은 PostgreSQL을 기준으로 관리하고, 재기동 시 DB 상태에서 처리 흐름을 복구합니다.
- Bloom은 같은 JVM의 Domain Event를 전달하고 Flower의 대기 Step과 Application Component를 연결합니다.
- Worker Chat에서 시작한 작업도 별도 우회 경로를 만들지 않고 기존 리포트·검토·문서 생성 Service와 Flower Flow를 사용합니다.

## AWS 운영 구성

현재 공개 사이트와 업무 Web, 관리자 Web, Cloud API, PostgreSQL, S3 호환 Storage 및 Cloud-managed Agent를 AWS Lightsail의 Docker 환경에서 운영하고 있습니다.

- Caddy 기반 HTTPS와 Domain Routing
- Nginx 기반 공개 사이트·사용자 Web·관리자 Web 제공
- Spring Boot Cloud API와 PostgreSQL
- MinIO/S3 호환 사진·문서 Object Storage
- Cloud-managed Agent의 문서 생성과 Artifact 전송
- 모델 API를 Provider Policy, 사용 한도와 호출 기록 계층 뒤에서 실행
- 법령 동기화와 운영 리포트의 예약 실행

## Repository 모듈

| 모듈 | 역할 |
|---|---|
| `cloud-api` | Tenant, 인증, 감리 업무 API, Engine/MCP, 법령, 문서 Job과 Workflow 관리 |
| `archdox-worker` | 정책·승인·Trace를 거쳐 Domain Action을 실행하는 통제 계층 |
| `archdox-ai-harness` | 감리 문서 검토와 문장 보정 등 ArchDox 전용 AI 업무 |
| `archdox-agent` | 현재 Local/Cloud 문서·사진·Artifact·Storage 실행 Server이며 Local AI Agent와 Agent MCP로 확장 예정 |
| `document-engine` | Template Binding과 HTML·DOCX·PDF 생성 기능 |
| `domain-shared` | 여러 모듈이 공유하는 Domain 값 |
| `client/web` | 사용자용 React Web |
| `admin` | 사무소·Platform 운영 관리자용 React Web |

## 개발 및 검증

- Gradle Multi-module Build와 Java 21
- Backend Compile 및 JUnit Test
- 사용자·관리자 React Web Build와 Vitest
- Docker Compose 구성 검증
- Agent DOCX·PDF 변환 Smoke Test
- MCP 도구와 Engine 검토 경계 Test
- GitHub Actions CI

## 관련 페이지

- [ArchDox 실제 서비스](https://archdox.co.kr/)
- [ArchDox 공개 저장소](https://github.com/parkKevinSB/archdox)
- [Flower 오픈소스]({{ '/projects/flower/' | relative_url }})
- [Flower AI Harness]({{ '/projects/flower-ai-harness/' | relative_url }})
- [Flower Action Runtime]({{ '/projects/flower-action-runtime/' | relative_url }})
- [Bloom Event Bus]({{ '/projects/bloom/' | relative_url }})

<p class="scope-note">이 페이지는 공개 저장소의 실제 코드, 현재 상태 문서와 운영 중인 공개 서비스를 기준으로 작성했습니다. 고객 데이터, 접속 정보와 내부 운영 설정은 포함하지 않습니다.</p>
