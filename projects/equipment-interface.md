---
title: 두께 측정기 CIM 연동 및 MQTT 검증
description: OLED 두께 측정기 제어 프로그램의 CIM 통신 연동과 MQTT 장비 제어 검증을 수행했습니다.
permalink: /projects/equipment-interface/
period: 2022.03 - 2022.07
category: 생산 장비 통신 연동 및 검증
role: 기존 프로그램 개조, CIM 통신 연동 및 MQTT 검증
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
