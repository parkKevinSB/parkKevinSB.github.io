---
title: 비전 정렬 및 검사 프로그램
description: 카메라 영상으로 위치와 각도를 계산하고 생산 레시피, 결과 데이터, 이미지와 알람 이력을 관리하는 프로그램입니다.
permalink: /projects/vision-inspection/
period: 2020.07 - 2022.12
category: 머신 비전 및 장비 제어
role: 응용 프로그램 개발, 기존 장비 개조 및 셋업
stack: C#, WPF, WinForms, GigE Camera, SQLite, Socket, Serial, ADS
---

## 비전 정렬 프로그램

- GigE 카메라 영상 취득
- 대상 물체의 위치와 각도 계산
- 계산한 보정값을 하위 Stage로 전달
- 생산 Recipe 생성·변경·적용
- 정렬 결과와 측정 데이터 저장
- 촬영 이미지, 로그와 알람 이력 조회
- Socket, Serial, ADS와 PLC 인터페이스 연동

<div class="image-grid">
  <figure class="evidence-figure">
    <img src="{{ '/assets/images/projects/vision-align.png' | relative_url }}" alt="비전 정렬 프로그램 화면">
    <figcaption>정렬 결과와 장비 상태 화면</figcaption>
  </figure>
  <figure class="evidence-figure">
    <img src="{{ '/assets/images/projects/vision-recipe.png' | relative_url }}" alt="비전 프로그램 레시피 관리 화면">
    <figcaption>생산 Recipe 관리 화면</figcaption>
  </figure>
</div>

## 데이터 관리

| 데이터 | 관리 내용 |
|---|---|
| Recipe | 제품별 카메라·정렬·검사 조건 |
| Result | 측정값, 보정값, 판정 결과 |
| Image | 작업별 촬영 이미지 |
| Alarm | 장비 및 검사 오류 이력 |
| Log | 통신과 작업 진행 기록 |

## 검사 장비 개조

OLED Mask 이장기와 검사기의 기존 C# WinForms 프로그램을 개조하고 현장 셋업을 수행했습니다.

- 1EA Mask Frame 인식기를 4EA 구성으로 변경
- BNR I/O와 Serial 통신
- ACS Gantry 제어 및 Teaching
- Review·Align Camera Teaching

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/mask-inspection.png' | relative_url }}" alt="OLED Mask 이장기 및 검사기 화면">
  <figcaption>OLED Mask 이장기 및 검사기 운영 화면</figcaption>
</figure>
