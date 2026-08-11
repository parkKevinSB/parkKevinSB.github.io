---
title: 비전 정렬 및 검사 프로그램
description: FLIR 산업용 카메라 영상으로 위치와 각도를 계산하고, 보정값을 Mitsubishi PLC가 제어하는 하위 Stage에 전달하는 C# 프로그램입니다.
permalink: /projects/vision-inspection/
period: 2020.07 - 2021.03
category: 머신 비전 및 장비 제어
role: 비전 정렬·검사 응용 프로그램 개발 및 장비 셋업
stack: C#, .NET Framework, WPF, SQLite, FLIR Camera, GigE Vision, Mitsubishi PLC, MX Component
---

## 프로젝트 개요

생산 대상의 위치와 각도를 카메라 영상으로 계산하고, 보정값을 하위 Stage에 전달하는 C# 기반 머신 비전 프로그램입니다. FLIR 산업용 카메라의 영상을 GigE Vision 인터페이스로 취득했으며, 하위 Stage는 Mitsubishi PLC가 제어했습니다.

제품별 촬영·정렬·검사 조건은 Recipe로 관리하고, 정렬 결과와 측정값, 알람 이력은 SQLite 기반 로컬 데이터베이스에 저장했습니다. 촬영 이미지와 작업 로그를 함께 남겨 생산 결과를 다시 확인할 수 있도록 구성했습니다.

## 담당 범위

- C#·WPF 기반 비전 정렬 및 검사 화면 개발
- FLIR 카메라 영상 취득과 정렬 알고리즘 적용
- 대상 물체의 위치·각도 계산과 Stage 보정값 산출
- Mitsubishi PLC가 제어하는 하위 Stage 연동
- SQLite 기반 Recipe·결과·알람 이력 관리
- 촬영 이미지와 통신·작업 로그 저장 및 조회
- 장비 연동 시험과 현장 셋업

## 정렬 및 검사 흐름

1. SQLite에서 생산 Recipe와 카메라·정렬·검사 조건을 불러옵니다.
2. FLIR 카메라 영상을 GigE Vision 인터페이스로 취득합니다.
3. 영상에서 대상 물체의 위치와 각도를 계산하고 기준값과 비교합니다.
4. 계산 결과를 하위 Stage 좌표계의 보정값으로 변환합니다.
5. MX Component를 통해 Mitsubishi PLC의 지정된 디바이스 영역에 보정값과 실행 상태를 전달합니다.
6. 정렬 결과, 측정값과 판정 결과를 SQLite에 저장하고 촬영 이미지·알람·작업 로그를 연결합니다.

## 카메라와 PLC 연동

| 구분 | 적용 내용 |
|---|---|
| 카메라 | FLIR 산업용 카메라 |
| 영상 인터페이스 | Gigabit Ethernet 기반 GigE Vision |
| 하위 Stage 제어 | Mitsubishi PLC |
| PC·PLC 통신 | MX Component의 .NET 통신 Control 사용 |
| 데이터 교환 | 양쪽에서 약속한 PLC 디바이스 영역을 통해 보정값과 실행 상태 전달 |
| 기타 장비 인터페이스 | 장비 구성에 따른 Socket·Serial 통신 |

MX Component가 C# 프로그램과 PLC에 하나의 공유 메모리를 만드는 방식은 아닙니다. C# 프로그램이 MX Component를 통해 PLC의 지정된 디바이스 값을 읽고 쓰며, 해당 영역을 보정값과 제어 상태를 교환하는 인터페이스로 사용했습니다.

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
| Recipe | SQLite에 제품별 카메라·정렬·검사 조건 저장 |
| Result | SQLite에 측정값, 보정값과 판정 결과 저장 |
| Image | 작업별 촬영 이미지 |
| Alarm | SQLite에 장비 및 검사 오류 이력 저장 |
| Log | 통신과 작업 진행 기록 |
