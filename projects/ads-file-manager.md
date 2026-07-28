---
title: TwinCAT ADS 파일 관리자
description: Beckhoff PLC의 장비 진행 상태를 기록하고 파라미터를 파일로 관리하는 상시 실행 프로그램입니다.
permalink: /projects/ads-file-manager/
period: 2021.03 - 2021.06
category: PLC 연동 및 운영 도구
role: 전체 프로그램 개발
stack: C#, WPF, TwinCAT ADS, XML, Socket, Tray Application
---

## 주요 기능

- Beckhoff PLC와 ADS 인터페이스 통신
- 장비 진행 상태 상시 Logging
- PLC Parameter 조회 및 파일 저장
- XML 기반 설정값 입출력
- Tray Icon 형태의 백그라운드 실행
- Socket을 통한 원격 Log·Parameter 관리
- 프로그램 연결 상태 및 오류 표시

<figure class="evidence-figure">
  <img src="{{ '/assets/images/projects/ads-file-manager.png' | relative_url }}" alt="TwinCAT ADS 파일 관리자 화면">
  <figcaption>TwinCAT ADS 기반 로그 및 파라미터 관리 프로그램</figcaption>
</figure>

## 프로그램 구성

| 구성 | 내용 |
|---|---|
| ADS 연결 | PLC Symbol과 상태 데이터 조회 |
| 로그 | 장비 상태 변경과 진행 기록 파일 생성 |
| 파라미터 | PLC 설정값 읽기·쓰기와 XML 저장 |
| 원격 관리 | Socket을 통한 로그와 파라미터 조회 |
| 실행 방식 | Windows 시작 시 자동 실행되는 Tray 프로그램 |
