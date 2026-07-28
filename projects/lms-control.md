---
title: LMS 물류 제어
description: 리니어 모터 스테이지와 주변 장치를 제어하는 PC 프로그램, Beckhoff 통신 라이브러리와 데모 장비 프로그램을 개발했습니다.
permalink: /projects/lms-control/
period: 2019.06 - 2021.09
category: PC 제어 및 PLC 제어
role: 프로그램 개발, 장비 개조 및 현장 셋업
stack: C#, WinForms, Beckhoff ST, Socket, Serial, CC-Link, XML
---

## C# 물류 제어 프로그램

- 상위 레이저 장비의 작업지시에 따른 LMS Stage 제어
- 복수 Stage의 작업 순서와 상태 관리
- 실린더, 센서, 조명 컨트롤러 연동
- 장비 상태와 알람 표시
- Socket, Serial, CC-Link 통신 적용
- 약 35대 장비 프로그램 개조 및 현장 셋업

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/lms-control.png' | relative_url }}" alt="LMS 물류 제어 프로그램 화면">
  <figcaption>기존 포트폴리오에 정리한 LMS 물류 제어 프로그램 화면</figcaption>
</figure>

## Beckhoff LMS 제어 라이브러리

기존 PC 제어 프로그램의 LMS 통신 기능을 Beckhoff PLC에서 사용할 수 있도록 ST 언어로 구현했습니다.

- Rexroth LMS Socket 인터페이스 분석
- Beckhoff용 Socket 통신 라이브러리 작성
- 동시 실행 제어를 위한 TestAndSet 적용
- 이벤트 기반 장비 제어 흐름 구현
- LMS Stage 데모 장비 프로그램 개발

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/beckhoff-lms.png' | relative_url }}" alt="Beckhoff LMS 제어 라이브러리 및 데모 장비">
  <figcaption>Beckhoff PLC와 LMS 데모 장비 연동 화면 및 시험 장비</figcaption>
</figure>

## 통신 구성

| 대상 | 통신 |
|---|---|
| 상위 장비 - 제어 프로그램 | Socket |
| LMS Stage | Socket |
| 센서·조명 컨트롤러 | Serial |
| 장비 입출력 | CC-Link |
| Beckhoff PLC | ST 기반 Socket Library |
