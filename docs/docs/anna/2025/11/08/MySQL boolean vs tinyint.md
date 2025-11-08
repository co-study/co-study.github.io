---
layout: default
parent: 안나 스터디
title: Mysql boolean vs tinyint
date: 2025-11-08
author: "조안나"
category: anna
tags:
  - MySQL
---

## 배경

MySQL에서 boolean으로 컬럼을 생성했을 때 Dbeaver에서 생성해주는 테이블 생성 DDL에는 Tinyint로 찍히는 걸 발견했습니다. 이 현상의 원인에 대해 알아보겠습니다.

## 원인

Mysql 내부에는 boolean 타입이 존재하지 않습니다. boolean = tinyint(1)의 동의어로, boolean으로 선언한 컬럼이 tinyint로 저장되는 것은 이 때문입니다. boolean은 인터페이스 상에서만 존재하며, 컬럼 타입에 대한 사용자의 의도를 더 잘 드러내기 위해 사용됩니다.

## tinyint

Mysql의 tinyint는 1바이트 값으로 -128 ~ 127 사이의 값을 저장할 수 있습니다 (UNSIGNED의 경우 0 ~ 255).
관례적으로 boolean 값을 나타낼 때 tinyint(1)라는 표현을 사용하는데, 괄호 안의 숫자는 사이즈와 관련이 없고 표시 폭을 나타내는 값입니다. 이 역시 현재는 deprecated 된 개념으로 큰 의미는 없습니다.

#### reference

https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html
https://stackoverflow.com/questions/11167793/boolean-or-tinyint-confusion
