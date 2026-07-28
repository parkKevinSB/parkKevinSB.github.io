---
title: 측정·검사 장비 연동
description: 두께 측정기 CIM 연동, MQTT 장비 제어 검증과 OLED Mask 장비 개조·셋업을 수행했습니다.
permalink: /projects/equipment-interface/
period: 2022.03 - 2022.12
category: 생산 장비 연동 및 현장 셋업
role: 기존 프로그램 개조, 통신 연동, 검증 및 셋업
stack: C#, WinForms, Socket, RS-232/485, CIM, MQTT, Node-RED
---

## 두께 측정기 CIM 연동

- 하위 Stage의 두께 측정기 실시간 제어
- Socket과 RS-232·485 Serial 통신
- 상위 CIM 시스템과 측정 결과 연동
- 기존 C# WinForms 프로그램 개조
- 장비 및 상위 시스템 연동 셋업

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/measurement-cim.png' | relative_url }}" alt="두께 측정기 CIM 연동 프로그램 화면">
  <figcaption>두께 측정기 제어 및 CIM 연동 화면</figcaption>
</figure>

## MQTT 검증

- Arduino와 Wi-Fi 모듈 구성
- MQTT 통신을 이용한 장비 On/Off 제어
- Node-RED를 이용한 시험 제어 화면 작성
- 모바일 및 PC 브라우저 제어 검증

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/mqtt-test.png' | relative_url }}" alt="MQTT 장비 제어 검증 보드">
  <figcaption>MQTT 통신 시험용 보드와 검증 구성</figcaption>
</figure>

## OLED Mask 장비

- Mask Frame 인식기 확장
- 장비 I/O 및 Serial 통신
- ACS Gantry 제어
- Review·Align Camera Teaching
- 기존 프로그램 개조 및 현장 셋업
