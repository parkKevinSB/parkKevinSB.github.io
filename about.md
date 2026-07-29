---
layout: default
title: 경력
description: KevinPark의 근무 이력, 담당 업무와 기술 경험을 정리합니다.
permalink: /about/
---

<section class="page-hero shell">
  <p class="section-kicker">경력</p>
  <h1>경력 및 기술</h1>
  <p>근무 이력과 담당 업무, 프로젝트에서 사용한 기술을 사실 중심으로 정리했습니다.</p>
</section>

<section class="shell prose-page" markdown="1">

## 근무 이력

| 기간 | 소속 | 담당 |
|---|---|---|
| 2023.01 - 현재 | 토탈소프트뱅크 연구소 · 대리 | 항만 자동화 시스템 유지보수, 신규 사이트 개발, 장비 에뮬레이터 및 검증 도구 개발 |
| 2019.06 - 2022.12 | 에이치아이티 제어팀 · 사원 | C# PC 제어, 머신 비전, Beckhoff PLC 제어, 장비 개조 및 현장 셋업 |
| 2016.09 - 2017.03 | 개인 프로젝트 | 주유소 정보 웹·앱 개발 및 데이터베이스 관리 |

## 토탈소프트뱅크

- 국내 2개, 일본 1개, 중동 1개 터미널의 ATCSS 운영 유지보수
- 일본 신규 터미널의 사이트별 기능 개발, ECS 에뮬레이터 검증, 배포 및 롤아웃
- 대만 항만 AYTSS 프로젝트의 기반 구조와 초기 버전 신규 개발
- TOS 작업지시를 FMS 실행 단위로 변환하는 WorkOrder 및 작업 흐름 구현
- 독자 개발한 `eventflow` 실행 프레임워크를 AYTSS에 최초 적용
- TLS [QMS]({{ '/projects/tls-qms/' | relative_url }}) Java 서버 설계·개발과 STS ECS 연동
- 선박 WorkQueue·WorkOrder 상태 관리, Single/Dual Trolley Scheduling과 작업 진행 보고 처리
- Netty 기반 [LI Net Socket]({{ '/projects/li-net-socket/' | relative_url }}) 구조 설계와 Spring Boot Starter·Agent Skill 개발
- ECS·STS 장비 에뮬레이터와 연동 검증 도구 설계 및 구현

## 에이치아이티

- Bosch LMS 물류 제어 프로그램 개발 및 약 35대 장비 개조·셋업
- 머신 비전 정렬 프로그램과 레시피·결과 데이터 관리 기능 개발
- Beckhoff PLC용 LMS 통신 라이브러리와 데모 장비 프로그램 개발
- TwinCAT ADS 기반 장비 상태 로거와 파라미터 파일 관리자 개발
- OLED 두께 측정기 CIM 연동 및 Mask 검사 장비 개조·셋업
- MQTT 기반 장비 제어 검증 코드 작성

## 개인·오픈소스 개발

- Flower를 기반으로 Slack 요청, 현장 로그·등록 DB 조회·소스 분석과 보고를 연결한 [사이트 유지보수 에이전트 서버]({{ '/projects/site-maintenance-agent/' | relative_url }})
- Java 단일 JVM 오케스트레이션 런타임 [Flower](https://github.com/flowerjvm/flower)
- Java 소스의 잘못된 Flower 사용을 19개 규칙으로 검사하는 `flower-check`
- AI 호출의 검증·재시도 수명주기를 관리하는 [Flower AI Harness](https://github.com/flowerjvm/flower-ai-harness)
- 업무 Action의 정책·승인·감사를 통제하는 [Flower Action Runtime](https://github.com/flowerjvm/flower-action-runtime)
- AI 코딩 에이전트용 [Flower App Guide](https://github.com/flowerjvm/flower-agent-skills)와 [Flower Action Runtime Guide](https://github.com/flowerjvm/flower-action-runtime-guide)
- JVM 내부 Typed Event Bus [Bloom](https://github.com/flowerjvm/bloom)
- 건축사사무소 문서 Workflow Platform [ArchDox](https://github.com/parkKevinSB/archdox)
- Maven Central 배포, GitHub Actions CI, 테스트와 공개 문서 관리

## 기술 경험

| 구분 | 기술 |
|---|---|
| 백엔드 | Java, Spring Boot, Spring Framework, eGovFrame, MyBatis, Spring JDBC |
| 오픈소스·프레임워크 | Flower, Bloom, AI Harness, Action Runtime, Maven Central, JUnit 5 |
| 개발 도구 | flower-check, JavaParser, SARIF, Agent Skills, Maven·Gradle Plugin |
| AI 연동 | Spring AI, OpenAI Java SDK, Anthropic Java SDK, Structured Output 검증 |
| 프론트엔드 | React, TypeScript, Vite, React Query |
| PC·장비 제어 | C#, .NET Framework, WinForms, WPF |
| 데이터베이스 | Oracle, MSSQL, MySQL, SQLite |
| PLC·산업 통신 | Beckhoff ST, TwinCAT ADS, CC-Link, RS-232/485 |
| 네트워크 | TCP Socket, 비동기 통신, MQTT |
| 머신 비전 | GigE Camera, 영상 정렬 및 검사, 레시피·결과 관리 |
| 설계 | 작업 시퀀스, 이벤트 기반 처리, 상태 관리, 스케줄링, 재시도·복구 |

## 학력 및 기타

- 한국기술교육대학교 메카트로닉스공학부
- 사무자동화산업기사
- GitHub: [parkKevinSB](https://github.com/parkKevinSB)

</section>
