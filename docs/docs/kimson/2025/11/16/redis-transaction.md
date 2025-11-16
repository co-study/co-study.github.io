---
layout: default
parent: 경남 스터디
title: 레디스 원자성 확보를 위한 분산락 처리
date: 2025-11-16
author: 김경남
category: kimson
tags:
  - Redis
  - LuaScript
---

# 레디스 원자성 확보를 위한 분산락 처리

레디스는 노드와 유사하게 싱글 스레드로 동작한다. 단일 레디스 노드를 구축해 사용해도 동시성 문제는 발생하지 않는다.

따라서 리소스에 대해 값이 설정된 경우 다른 리소스 접근을 차단할 수 있다. 이를 잠금이라 표현하고, 이를 다음과 같이 명령을 사용해 재현할 수 있다.

## Lock(락)의 개념

"락을 회득한다"라는 말을 자주 사용하게 되는데, 공유 자원을 혼자 안전하게 사용하기 위해 다른 클라이언트가 접근하지 못하게 점유권을 빌리는 것을 말한다.

즉, 재고가 10일 경우, A와 B가 서로 수정하려 할 때 한 순간에 오직 하나의 클라이언트만 재고 수정 가능하게 만드는 장치를 락이라 한다.

## 락 유무에 따른 시나리오

자원: stock:drink001 (현 재고: 10)

- clientA: 재고 +5 증가 예정
- clientB: 재고 -3 감소 예정
- 동시에 요청 발생

만약 두 요청이 GET/SET 한다면?

### 락 없는 경우

```bash
# clientA 수정 시도
GET stock:drink001 # -> 10
# 계산: 10 + 5 = 15
SET stock:drink001 15
```

```bash
# clientB 수정 시도 (거의 동시 수행 가정하고 서로 GET한 시점이 같을 때)
GET stock:drink001 # -> 10 (clientA SET 전)
# 계산: 10 - 3 = 7
SET stock:drink001 7
```

이 경우 최종 재고가 7이 되고, clientA의 작업이 완전 날아가버린다. 이를 경쟁 조건(Race Condition)이라한다. 이 때문에 공유 자원을 두고 안정하게 데이터를 사용하거나 수정하기 위해 락이 필요하다.

## 락 획득(Acquire the lock)

레디스에서는 아래와 같이 표현 된다.

```bash
SET lock:stock:drink001 <UUID> NX EX 10
```

이 한 줄이 "락 획득"이 된다.

락 획득의 조건은 다음과 같다.

### 락 획득 조건

- NX: 락 키가 없을 때만 생성
- EX 10: 락은 10초 후 자동 제거
- «UUID»: 락 소유자를 식별하는 일종의 신분증 역할(인증 토큰 개념)

### 락 획득 성공 조건

누구든 먼저 이 명령을 성공하면 그 자원을 점유하는 상태가 된다.

```bash
SET lock:stock:drink001 C1_UUID NX EX 10
# 성공 -> clientA가 락을 획득
```

```bash
SET lock:stock:drink001 C2_UUID NX EX 10
# 실패 -> 해당 락키가 있기 때문에 접근 불가
```

## 락 획득 후 달라지는 점

clientA는 이제 다음처럼 안전하게 데이터 변경 가능

```bash
# 현재 재고
GET stock:drink001 # 10
# 계산
SET stock:drink001 <10 + n>
```

작업 이후 락을 풀어줘야 하는데 아래와 같이 락을 풀게 된다.

```bash
DEL lock:stock:drink001
```

여기서 문제가 발생하는데 client가 요청할 때 동시성 문제로 인해 아래와 같은 시나리오라면 큰 문제가 발생할 수 있다.

1. clientA가 락 획득
2. 어떤 타이밍에 clientB가 다시 락 획득
3. clientA가 늦게 작업 된 경우 DEL 명령
4. clientB의 락 해제

위와 같은 시나리오는 "lock:" 키를 설정할 때 만료시간 보다 더 긴 작업이 있을 경우 충분히 발생할 수 있는 문제다.

때문에 단순히 DEL 명령으로 제거하면 안되고, 해당 lock키의 uuid 값이 유효한지 검증한 후 제거해야한다.

## 레디스의 트랜젝션

레디스는 MySQL과 같은 SQL 계열의 DB와 달리 다른 의미의 트랜젝션을 수행한다. SQL의 트랜젝션과 같이 여러 개의 명령을 하나의 묶음으로 만들어 순서대로 실행하고, 명령 순서가 보장된다.

레디스에서 롤백 기능이 없고 SQL처럼 예외 발생 시 데이터를 되돌리는 기능이 없다.
일부 WATCH 기반의 낙관적 락에서는 예외이다. 때문에 원자성이 완벽하게 보장되지 않는 문제가 발생한다.

레디스에서 제공하는 트랜젝션은 3가지 명령이 있는데, MULTI, WATCH, EXEC이며, 3가지 명령을 조합해서 사용하더라도 락 획득 실패하거나 데이터 수정 실패 시 실패한 요청의 경우 모든 프로세스가 정지되는 현상이 발생한다.

## 레디스 원자적 작업

레디스에서는 lua 스크립트 실행을 지원한다. 위 설명했던 트랜젝션을 구현하는데 단점을 보완하여 lua 스크립트를 활용하면 훨씬 강력하고 안정한 트랜젝션을 구현할 수 있다.

lua 스크립트는 레디스 내부에서 스크립트를 한 명령처럼 실행하기 때문에 "조건 확인 + 업데이터 + 삭제" 전체가 "불가분 원자적"이다.

락 해제, 재고감소와 같은 복잡한 로직에 필수적으로 사용된다.

```typescript
async function veryHeavyLogic() {
	const lockKey = "lock:stock:drink001";
	const stockKey = "stock:drink001";
	const lockUUID = nanoid(16);
	const calcValue = 5;

	await redis.eval(`
		local lockKey = KEY[1]
		local stockKey = KEY[2]
		local lockUUID = ARGV[1]
		local value = ARGV[2]

		if redis.call('GET', lockKey) == lockUUID then
			redis.call('SET', stockKey, value)
			redis.call('DET', lockKey)
		end
	`, lockKey, stockKey, 2, lockUUID, calcValue);
}
```

위와 같이 lua 스크립트를 활용하면 스크립트 한 묶음을 하나의 명령처럼 처리하기 때문에 안전하게 원자적으로 데이터를 점유하고 수정가능하게 된다.

다른 클라이언트와 동시에 수정하더라도 다른 클라이언트의 요청은 실패하고 안전하게 데이터 수정이 용이하게 된다.

실패한 요청은 큐를 이용해 재시도하거나 버리는 식의 처리가 가능해진다.

## 마무리

레디스를 단순히 Key-Value 쌍의 인메모리 DB로 생각하고 간단한 세션 관리나 소켓과 유사하게 브로드캐스트하는 용도로만 생각했지만, 트랜젝션 관리와 lua 스크립트 결합을 통해 동시성에 관한 기술적인 이해도를 높인 좋은 기회였다.

오늘 다룬 내용은 분산락에 대한 개념이자, 레디스가 제공하는 분산락(레드락)의 한계를 lua 스크립트로 보완하는 내용을 주로 다룬다. 앞으로 락에 관한 키워드로 낙관락, 비관락을 이해하고 더 공부해야겠다는 생각이 든다.

---

## 레퍼런스

- [레디스가 제공하는 분산락(RedLock)](https://mangkyu.tistory.com/311)
- [node.js 환경에서 redis 분산락 구현](https://growth-coder.tistory.com/319)

---

📅 **작성일:** 2025-11-16 11:45:03\
✍️ **작성자:** 김경남
