---
title: LMS 물류 제어 프로그램
description: Bosch Rexroth LMS Stage와 주변 장치를 제어하는 C# 물류 제어 프로그램을 개발하고 장비 개조·현장 셋업을 수행했습니다.
permalink: /projects/lms-control/
period: 2019.06 - 2020.07
category: PC 기반 물류 장비 제어
role: C# 제어 프로그램 개발, 장비 개조 및 현장 셋업
stack: C#, WinForms, Socket, Serial, CC-Link
---

## 프로젝트 개요

Bosch Rexroth LMS Stage를 이용하는 물류 장비의 C# 제어 프로그램을 개발했습니다. 상위 레이저 장비의 작업지시를 받아 LMS Stage와 주변 장치를 제어하고, 약 35대의 장비 프로그램 개조와 현장 셋업을 수행했습니다.

## 주요 개발 내용

- 상위 레이저 장비의 작업지시에 따른 LMS Stage 제어
- 복수 Stage의 작업 순서와 상태 관리
- ESC·조명 컨트롤러와 실린더·센서 연동
- 장비 상태와 알람 표시
- Socket, Serial, CC-Link 통신 적용
- 약 35대 장비 프로그램 개조 및 현장 셋업

<figure class="evidence-figure evidence-figure-compact">
  <img src="{{ '/assets/images/projects/lms-control.png' | relative_url }}" alt="LMS 물류 제어 프로그램 화면">
  <figcaption>Bosch Rexroth LMS 물류 제어 프로그램 화면</figcaption>
</figure>

## 통신 구성

| 대상 | 통신 |
|---|---|
| 상위 레이저 장비 - 제어 프로그램 | Socket |
| Bosch Rexroth LMS Stage | Socket |
| ESC·조명 컨트롤러 | Serial |
| 실린더·센서 | CC-Link |

## 관련 프로젝트

- [Beckhoff LMS 제어 라이브러리]({{ '/projects/beckhoff-lms-library/' | relative_url }}) — Bosch Rexroth LMS를 Beckhoff PLC에서 제어하기 위한 ST 통신 라이브러리와 데모 프로그램
