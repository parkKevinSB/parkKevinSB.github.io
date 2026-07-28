---
title: ArchDox
description: 건축사사무소의 현장 점검·감리·사진 관리·문서 생성과 운영 업무를 처리하는 문서 Workflow Platform입니다.
permalink: /projects/archdox/
period: 2025 - 현재
category: 개인 개발 · 공개 저장소
role: 전체 Architecture 및 Backend·Frontend·문서 실행 구조 개발
stack: Java 21, Spring Boot, Gradle, PostgreSQL, MinIO/S3, React, TypeScript, Docker, Flower, Bloom
---

## 프로젝트 개요

ArchDox는 건축사사무소의 현장 점검, 감리, 사진 정리, 문서 생성과 운영 업무를 하나의 작업 흐름으로 관리하는 Platform입니다. Cloud API, 사용자 Web, 관리자 Web, 문서 실행 Agent와 Workflow 모듈을 하나의 저장소에서 개발하고 있습니다.

| 항목 | 내용 |
|---|---|
| 공개 저장소 | [github.com/parkKevinSB/archdox](https://github.com/parkKevinSB/archdox) |
| 개발 상태 | 개발 진행 중 |
| Backend | Java 21, Spring Boot 3.5, Gradle |
| Frontend | React 19, TypeScript, Vite |
| Infrastructure | PostgreSQL, MinIO/S3, Docker, LibreOffice |

## 시스템 구성

| 모듈 | 역할 |
|---|---|
| `cloud-api` | Tenant, 인증, REST API, Workflow, Storage와 문서 Job 관리 |
| `archdox-agent` | 문서 변환, 사진 수집, Artifact와 Storage 작업 실행 |
| `document-engine` | 문서 생성을 위한 재사용 가능한 처리 기능 |
| `archdox-ai-harness` | 문서 사전 검토, QA와 운영 진단 등 ArchDox 전용 AI 업무 |
| `archdox-worker` | 정책·승인·Trace를 거쳐 Domain Action을 실행하는 통제 계층 |
| `domain-shared` | 여러 모듈에서 사용하는 Domain Enum과 값 |
| `client/web` | 사용자용 React Web |
| `admin` | 운영·관리자용 React Web |

## Backend 개발

- Tenant 단위 데이터 분리와 실행 Context
- Spring Security 기반 인증·권한 처리
- JPA와 PostgreSQL 기반 Domain 저장
- Flyway Database Migration
- MinIO 및 S3 호환 Object Storage
- WebSocket 기반 상태 전달
- 문서 Job, Artifact, 사진과 파일 처리
- PDFBox 기반 PDF 처리
- Agent 등록과 명령 Routing
- Actuator Health 및 운영 상태 확인

## ArchDox Agent

ArchDox Agent는 AI Agent가 아니라 문서와 파일 작업을 실행하는 등록형 실행 Server입니다.

- `LOCAL_OFFICE`와 `CLOUD_MANAGED` 실행 방식
- DOCX·PDF 등 문서 변환
- LibreOffice와 한국어/CJK Font가 포함된 Docker Image
- 사진 수집과 Artifact Upload·Download
- S3 호환 Storage 접근
- Cloud API의 작업 명령 수신과 결과 보고

## Flower 기반 실행 구조

### archdox-worker

- 사용자·API·AI가 요청한 Domain Action을 동일한 통제 절차로 처리
- 권한, Policy, 승인과 Trace 확인
- 허용된 Action만 Domain Service로 전달
- 장시간 작업을 Flower Flow로 실행

### archdox-ai-harness

- 범용 Flower AI Harness 위에 ArchDox 업무를 구현
- Report 사전 검토
- 문서 품질 확인
- Worker Conversation Planning
- 운영 상태 진단

### Bloom

- Module 사이의 JVM 내부 Domain Event 전달
- Flower의 대기 Step과 Application Component가 동일 Event를 구독할 수 있도록 Adapter 적용

## Frontend 개발

- 사용자용 React Single Page Application
- 운영·관리자용 별도 React Application
- React Router 기반 화면 구성
- React Query 기반 Server State 조회
- React Hook Form 기반 입력 처리
- Vitest 기반 Frontend Test

## 개발 및 검증 환경

- Gradle Multi-module Build
- Backend Compile 및 JUnit Test
- 사용자·관리자 Web Build
- Docker Compose 설정 검증
- PostgreSQL, MailHog와 MinIO Local 구성
- Agent PDF 변환 Smoke Test
- GitHub Actions CI

## 관련 페이지

- [Flower 오픈소스]({{ '/projects/flower/' | relative_url }})
- [Flower AI Harness]({{ '/projects/flower-ai-harness/' | relative_url }})
- [Flower Action Runtime]({{ '/projects/flower-action-runtime/' | relative_url }})
- [Bloom Event Bus]({{ '/projects/bloom/' | relative_url }})
- [Flower JVM 프로젝트 역할 구분]({{ '/notes/flower-jvm-project-boundaries/' | relative_url }})

<p class="scope-note">공개 저장소에는 개발 중인 코드와 문서가 포함되어 있습니다. 이 페이지는 현재 공개된 Repository의 모듈 구성과 구현 범위를 기준으로 작성했습니다.</p>

