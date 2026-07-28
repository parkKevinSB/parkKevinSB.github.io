---
title: 착한주유소
description: 전국 약 1만 2천 개 주유소의 위치와 변경·불법 이력 데이터를 조회할 수 있도록 만든 개인 웹·앱 프로젝트입니다.
permalink: /projects/good-gas-station/
period: 2016.09 - 2017.03
category: 개인 웹 개발
role: 데이터 수집·정리, DB 및 웹·앱 개발
stack: JavaScript, HTML, CSS, PHP, MySQL, GPS
---

## 프로젝트 개요

전국 주유소 정보를 수집해 위치와 이력을 확인할 수 있도록 구성한 개인 프로젝트입니다.

## 주요 개발

- 전국 약 1만 2천 개 주유소 데이터 관리
- 주유소 고유 코드, 주소와 좌표 데이터 정리
- 상호 및 운영 정보 변경 이력 관리
- 불법 행위 이력 데이터 연계
- GPS를 이용한 주변 주유소 검색
- 지도를 이용한 위치 표시
- PHP를 통한 MySQL 데이터 조회
- 웹 화면과 모바일 앱 형태의 조회 기능 구현

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/good-gas-station.png' | relative_url }}" alt="착한주유소 웹과 모바일 화면">
  <figcaption>주유소 검색, 지도와 상세 이력 조회 화면</figcaption>
</figure>

## 데이터 구성

| 데이터 | 내용 |
|---|---|
| 기본 정보 | 주유소 코드, 상호, 주소 |
| 위치 | 위도·경도와 지도 표시 |
| 변경 이력 | 상호와 운영 정보 변경 |
| 이력 정보 | 공개 데이터 기반 위반 이력 |
