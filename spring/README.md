# Spring 디자인 패턴 가이드

## 정의

Spring Framework 환경에서 자주 사용되는 디자인 패턴들을 실무 관점에서 정리한 가이드입니다. 기본 GoF 패턴을 Spring의 DI, AOP, Bean 관리 특성에 맞게 적용하는 방법을 다룹니다.

## 🎯 한눈에 보기

| 패턴 | 핵심 | Spring 활용 |
|------|------|-------------|
| [**Strategy**](./Strategy/README.md) | Enum + Map<Enum, Strategy> | List 주입 → Map 변환, Jackson 다형성 |

## 문서 가이드

| 문서 | 설명 | 난이도 |
|------|------|--------|
| [Spring 전략패턴](./Strategy/README.md) | Enum 기반 전략 선택, Jackson 다형성, 제네릭 인터페이스 | ⭐⭐ |

## 관련 문서

| 문서 | 설명 |
|------|------|
| [GoF 전략 패턴](../behavioral/Strategy/README.md) | 기본 전략 패턴 개념 |
| [TDD 가이드](../tdd/README.md) | 테스트 주도 개발 |
