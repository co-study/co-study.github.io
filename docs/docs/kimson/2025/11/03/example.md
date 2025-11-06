---
layout: default
parent: 경남 스터디
title: (예시) NestJS 의존성 주입 원리 이해하기
date: 2025-11-03
author: GPT5
category: kimson
tags:
  - NestJS
  - TypeScript
  - Dependency Injection
  - Backend
---

# 🧠 (예시) NestJS 의존성 주입 원리 이해하기

---

## 1️⃣ 어떤 주제로 공부했나요?

> 최근 NestJS로 프로젝트를 진행하면서 DI(Dependency Injection) 컨테이너 구조가 헷갈려서  
> 의존성 주입의 원리와 실제 동작 방식을 정리해보았습니다.

- NestJS의 DI가 어떻게 동작하는지 궁금했음
- Angular 기반 구조와의 유사점, 차이점이 있는지 확인하고 싶었음

---

## 2️⃣ 스터디 과정 (어떤 방식으로 공부했는가)

- NestJS 공식문서 및 DI 관련 챕터 정독  
  [NestJS Dependency Injection 공식문서](https://docs.nestjs.com/fundamentals/dependency-injection)
- 인프런 NestJS 강의 참고
- 팀원과 DI가 필요한 실제 사례를 들어 설계 토의
- 예시 실습: `@Injectable()`과 `ModuleRef`를 이용해 커스텀 Provider 작성

---

## 3️⃣ 알게 된 점

- NestJS의 DI는 Angular에서 발전된 구조 사용
- 의존성 주입이 클래스 생성자 레벨에서 TypeScript의 타입 정보와 Reflect Metadata 기반으로 동작
- Provider 등록 시 Lazy Loading, Scope 변환 등 다양한 옵션 제공
- 복잡한 의존 관계를 효율적으로 관리할 수 있음

---

## 4️⃣ 더 궁금한 점

- Request 스코프 Provider의 주입 타이밍 및 라이프사이클 관리
- 커스텀 Provider 작성 시 내부적으로 어떤 Hook이 동작하는지
- DI 에러가 발생했을 때 디버깅 노하우

---

## 5️⃣ 결과적으로 느낀 점

- DI를 제대로 이해하니 모듈화와 유지보수성이 높은 구조를 설계할 수 있을 것 같음
- NestJS의 내부 동작을 이해하는 데 도움이 되었음
- 다음 스터디에선 AOP 개념과 Interceptor를 연계해보고자 함

---

📅 **작성일:** 2025-11-03  
✍️ **작성자:** GPT5