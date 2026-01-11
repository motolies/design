# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

초급 자바 개발자를 위한 GoF 디자인 패턴 학습 가이드. 이론과 실제 예제를 함께 제공하는 교육용 문서 저장소.

## 프로젝트 구조

```
design/
├── creational/          # 생성 패턴 (Singleton, Factory Method, Builder, Abstract Factory, Prototype)
├── structural/          # 구조 패턴 (Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy)
└── behavioral/          # 행동 패턴 (Observer, Strategy, Template Method, Command, State, Chain of Responsibility)
```

## 문서 작성
- 모든 폴더에 `README.md` 포함
- 도표는 Mermaid 사용, 복잡한 경우 SVG 파일로 대체
- 실생활 예제를 적극 활용 (결제, 할인 시스템 등 구체적 시나리오)
- 사용자가 수정한 내용이 있으면 그 방향을 따름

## 코드 예제
- Java와 Spring 기반 예제
- 가장 심플한 단위로 작성
- JDK 21 (Amazon Corretto) 사용

## 각 패턴 문서 구조

```
- 정의
- 구조 (Mermaid UML)
- 사용 이유
- 적용 상황
- 실생활 예제
- 기본 예제 (Java/Spring)
- 장단점 분석
- 관련 패턴
```
