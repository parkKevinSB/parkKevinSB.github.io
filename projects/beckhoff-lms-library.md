---
title: Beckhoff LMS 제어 라이브러리
description: Bosch Rexroth LMS를 Beckhoff PLC에서 제어하기 위해 ST 기반 통신 라이브러리와 데모 장비 프로그램을 개발했습니다.
permalink: /projects/beckhoff-lms-library/
period: 2021.06 - 2021.09
category: PLC 제어 라이브러리 및 데모 장비
role: LMS 통신 인터페이스 이식, Socket 라이브러리와 ST 데모 프로그램 개발
stack: Beckhoff ST · Socket · TestAndSet · Observer Pattern
---

## 프로젝트 개요

LMS Stage는 Bosch Rexroth 장비입니다. 기존 Rexroth LMS 통신 인터페이스를 Beckhoff PLC에서 사용할 수 있도록 ST 기반 제어 라이브러리로 이식하고, 실제 LMS Stage를 제어하는 데모 프로그램을 개발했습니다.

| 구분 | 내용 |
|---|---|
| 제어 대상 | Bosch Rexroth LMS Stage |
| 제어 플랫폼 | Beckhoff PLC |
| 개발 목적 | Rexroth LMS 통신 기능을 Beckhoff 환경에서 사용 |
| 라이브러리 | ST 기반 Socket 통신 및 동시 실행 제어 |
| 데모 프로그램 | LMS 연결, 상태 반영과 Stage 제어 |

## 통신 구조

<div class="compact-process" aria-label="Beckhoff PLC와 Bosch Rexroth LMS 통신 구조">
  <span>Beckhoff PLC<br>ST Application</span><b>→</b><span>Beckhoff LMS<br>제어 라이브러리</span><b>→</b><span>Socket<br>통신</span><b>→</b><span>Bosch Rexroth<br>LMS Stage</span>
</div>

## 주요 개발 내용

- Rexroth LMS 통신 인터페이스 분석 및 Beckhoff용 라이브러리 이식
- Beckhoff ST 기반 Socket 통신 기능 구현
- TestAndSet을 적용한 동시 실행 제어
- 기존 C# LMS 제어 흐름을 ST 데모 프로그램으로 이식
- Observer Pattern을 이용한 장비 상태 반영
- 데모 장비에서 LMS 연결과 Stage 제어 검증

<figure class="evidence-figure evidence-figure-compact">
  <img src="{{ '/assets/images/projects/beckhoff-lms.png' | relative_url }}" alt="Beckhoff PLC와 Bosch Rexroth LMS 데모 장비">
  <figcaption>Beckhoff PLC용 LMS 제어 라이브러리와 데모 장비 — 공개용으로 원본 해상도를 축소했습니다.</figcaption>
</figure>

## 관련 프로젝트

- [LMS 물류 제어 프로그램]({{ '/projects/lms-control/' | relative_url }}) — C#으로 개발한 Bosch Rexroth LMS 물류 장비 제어 프로그램

<p class="scope-note">공개 페이지에는 포트폴리오에 정리된 라이브러리 역할과 개발 범위만 표시했으며, 통신 전문과 현장별 제어 조건은 포함하지 않았습니다.</p>
