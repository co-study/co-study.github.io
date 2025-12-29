---
layout: default
parent: 안나 스터디
title: SQL 전문가 가이드_ 인덱스 구조
date: 2025-12-28
author: 조안나
category: anna
tags:
  - NestJS
---

## 가. 인덱스 기본 구조

1. B-Tree 인덱스

![image.png](attachment:9d2b8985-8bbb-4cf3-b954-4963e1c958b9:image.png)

- 나무를 뒤집어 놓은 모양새. 맨 위가 root, 아래가 leaf이다
- 루트에서 리프 블록까지의 거리를 인덱스 깊이(Height)라고 부르며, 인덱스를 반복적으로 탐색할 때 성능에 영향을 미친다.
- 리프 블록은 인덱스 키 값과 그 키 값에 해당하는 테이블 레코드를 찾아가는데 필요한 주소정보(ROWID)를 갖는다. 키 값이 같을 때는 ROWID 순으로 정렬된다.
- 리프 블록은 항상 인덱스 키 값 순으로 정렬되어 있다. → 범위 스캔이 가능하다.
- 정방향과 역방향 스캔이 둘 다 가능하도록 양방향 연결 리스트 구조로 연결돼있다.

### null 값을 인덱스에 저장한다면?

- Oracle: 인덱스 구성 컬럼이 모두 null인 레코드는 인덱스에 저장하지 않는다. 하나라도 값이 있으면 저장하는데, null 값을 맨 뒤에 저장한다.
- SQL Server: 인덱스 구성 컬럼이 모두 null인 레코드도 인덱스에 저장한다. null 값을 맨 앞에 저장한다.

## 나. 인덱스 탐색

### 수평적 탐색 / 수직적 탐색

- 수평적 탐색: 리프 블록끼리 좌에서 우로 스캔하는 탐색.
- 수직적 탐색: 수평적 탐색을 위한 시작 지점을 찾는 과정으로, 루트에서 원하는 값을 가진 리프까지 탐색하는 것

# 2. 다양한 인덱스 스캔 방식

## 가. Index Range Scan

![image.png](attachment:a114dfd4-70f3-426c-ba21-4f6b07ec29ec:image.png)

- B-tree 인덱스의 가장 일반적이고 정상적인 형태의 액세스 방식

Oracle

```sql
... INDEX (RANGE SCAN) OF 'EMP_DEPTNO_IDX'(INDEX)
```

SQL Server

```sql
... Index Seek(OBJECT:([..].[dbo].[emp].[emp_deptno_idx]), SEEK:([deptno] = 20 ORDERED FORWARD)
```

- 인덱스를 수직적으로 탐색한 후에 리프 블록에서 ‘필요한 범위’만 스캔한다.
- 인덱스를 스캔하는 범위(Range)를 얼마만큼 줄일 수 있느냐, 테이블로 액세스하는 횟수를 얼마만큼 줄일 수 있느냐가 관건이다. 이는 인덱스 설계와 SQL 튜닝의 핵심 원리 중 하나이다.
- Index Range Scan이 가능하게 하려면 인덱스를 구성하는 선두 컬럼이 조건절에 사용되어야 한다. 그렇지 못한 상황에서 인덱스를 사용하도록 힌트로 강제하면, 바로 이어서 설명할 Index Full Scan 방식으로 처리된다.
- Index Range Scan 과정을 거쳐 생성된 결과 집합은 인덱스 컬럼 순으로 정렬된 상태가 되기 때문에, 이런 특성을 잘 이용하면 osrt order by 연산을 생략하거나, min/max 값을 빠르게 추출할 수 있다.

## 나. Index Full Scan

![image.png](attachment:37bf3a5e-8f1c-413a-a486-33793a6d669d:image.png)

- 수직적 탐색 없이, 인덱스 리프 블록을 처음부터 끝까지 수평적으로 탐색하는 방식이다. 대개는 데이터 검색을 위한 최적의 인덱스가 없을 때 차선으로 선택된다.
- 실제로는, 맨 첫 리프 노드까지 가기 위해 한번의 수직적 탐색이 일어나긴 한다 (그림의 점선)

### Index Full Scan의 효용성

- 인덱스 선두 컬럼이 조건절에 없으면, 옵티마이저는 우선적으로 Table Full Scan을 고려한다. 그런데 대용량 테이블이어서 Table Full Scan의 부담이 크다면, Index Full Scan을 고려한다.
- 데이터 저장공간은 가로 X 세로 즉 컬럼길이 * 레코드수 에 의해 결정된다. 대개 인덱스가 차지하는 면적은 테이블보다 훨씬 적다. 만약 인덱스 스캔 단계에서 대부분 레코드를 필터링하고 일부에 대해서만 테이블 액세스가 발생하는 경우라면, 테이블 전체를 스캔하는 것보다 낫다. 이럴 때 옵티마이저는 Index Full Scan 방식을 선택할 수 있다.

### 인덱스를 이용한 소트 연산 대체

- Index Full Scan은 Index Range Scan과 마찬가지로, 그 경로가 집합이 인덱스 컬럼 순으로 정렬되므로 Sort order By연산을 생략할 목적으로 사용될 수도 있다. (차선책 x 옵티마이저가 전략적으로 선택)

![image.png](attachment:d3fb2aec-5156-4115-992c-6f180d80ea63:image.png)

![image.png](attachment:737b8753-f96b-4325-bd28-fc8522aff65c:image.png)

- where sal > 1000
  - 대부분 사원 연봉이 5000을 넢어 Index Full Scan을 하면 거의 모든 레코드에 대해 테이블 액세스가 발생해 Table Full Scan보다 오히려 불리
  - 만약 sal이 인덱스 선두 칼럼이어서 Index Range Scan을 하더라도 마찬가지임
  - 그런데도 여기서 인덱스가 사용된 것은 사용자가 first_rows 힌트를 이용해 옵티마이저 모드를 바꾸었기 때문임 - 즉 옵티마이저가 소트 연산을 생략함으로써 전체 집합 중 처음 일부만을 빠르게 리턴할 목적으로 Index Full Scan 방식을 선택한 것

## 다. Index Unique Scan

- 수직적 탐색만으로 데이터를 찾는 스캔 방식
- Unique 인덱스를 ‘=’ 조건으로 탐색하는 경우에 작동

Oracle의 실행계획

![image.png](attachment:83eb2c69-9a8e-4e6b-8c61-60a85f4000e3:image.png)

- INDEX (UNIQUE SCAN)
- SQL Server는 Range Scan 과, Unique Scan을 구분하지 않고 똑같이 Index Seek으로 표시한다.

## 라. Index Skip Scan

![image.png](attachment:45817b24-3344-4e0a-8461-ad9d0667ca9a:image.png)

- 원래 인덱스 선두 컬럼이 조건절에 사용되지 않으면 → Table Full Scan, 또는 I/O를 줄일 수 있거나 정렬된 결과를 쉽게 얻을 수 있을 때 Index Full Scan인데
- 오라클 9i 버전부터 인덱스 선두 컬럼이 조건절에 빠졌어도 인덱스를 활용하는 새로운 스캔방식을 선보임. 그게 Index Skip Scan이다.

예) 성별 - 연봉 인덱스가 있는 테이블에서 인덱스 선두 컬럼인 성별이 조건절에 없는 쿼리 실행시

![image.png](attachment:59a01047-9343-4903-8dc9-c7588c078a1d:image.png)

- Skip Scan 의 원리: 루트 또는 브랜치 블록에서 읽은 칼럼 값 정보를 이용해 조건에 부합하는 레코드를 포함할 ‘가능성이 있는’ 하위 블록(브랜치 또는 리프 블록)만 골라서 액세스 (가능성을 어케 판단하지?)
- 이 스캔 방식은 조건절에 빠진 인덱스 선두 컬럼의 Distinct Value 개수가 적고, 후행 컬럼의 Distinct Value 개수가 많을 때 유용

![image.png](attachment:9fdf7fe4-b55c-4f7e-8173-f4901e16801b:image.png)

- INLIST ITERATOR
  - 조건절 In-List(라고 부르는구나?)에 제공된 값의 종류만큼, 인덱스 탐색을 반복 수행함을 뜻한다.
  - 이렇게 쿼리 작성자가 직접 성별에 대한 조건식을 추가해주면 Index Skip Scan에 의존하지 않고도 빠르게 결과 집합을 얻을 수 있다. 단 이처럼 In-List를 명시하려면 성별 값의 종류가 더 이상 늘지 않음이 보장되어야 한다. 그리고, In-List로 제공하는 값의 종류가 적어야 한다.
  - In-List를 제공하는 튜닝 기법을 익히 알던 독자라면 Index Skip Scan이 옵티마이저가 내부적으로 In-List를 제공해주는 방식이라고 생각하기 쉽지만 내부 수행 원리는 전혀 다르다.

## 마. Index Fast Full Scan

- 말 그대로, Index Full Scan보다 빠른 검색 방식이다.
- 이유는 인덱스 트리 구조를 무시하고 인덱스 세그먼트 전체를 Multiblock Read 방식으로 스캔하기 때문이다. Index Full Scan과의 차이점을 요약하면 다음과 같다:

![image.png](attachment:08922a7a-7e41-4a0d-a2e5-be4312957801:image.png)

## 바. Index Range Scan Descending

![image.png](attachment:8c6642d3-7d4f-4e72-aed8-c1a8c8e72f4f:image.png)

- Index Range Scan과 기본적으로 동일한 스캔 방식이다. 그림처럼 인덱스를 뒤에서부터 앞으로 스캔하기 때문에, 내림차순으로 정렬된 결과 집합을 얻는다는 점만 다르다.

예2 ) emp 테이블을 empno 기준으로 내림차순 정렬하는 쿼리이며, empno 컬럼에 인덱스가 있는 경우

![image.png](attachment:af1dc32f-a6ec-442c-ac7c-58b373601cde:image.png)

- 옵티마이저가 알아서 인덱스를 거꾸로 읽는 실행계획을 수립한다.

SQL Server버전

![image.png](attachment:c90e55b0-8c19-465f-90f1-e8f715ae817c:image.png)

예2 ) Max 값 구하기

![image.png](attachment:3f5a0bd4-4fe1-4b37-9e81-761333f32be7:image.png)

- Max 값을 구하려는 컬럼에 인덱스가 있으면, 인덱스를 뒤에서부터 한건만 읽고 멈추는 실행계획이 수립된다.
