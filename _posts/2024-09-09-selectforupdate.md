---
layout: post
title: select for update문
categories: DB
---

select for update문은 데이터베이스에서 동시성 제어를 위해 사용되는 SQL 구문으로, 트랜잭션에서 특정 레코드를 잠금으로써 다른 트랜잭션이 해당 레코드를 수정하지 못하도록 하는 기능이다.

select for update문을 실행하면 Lock을 획득하고, 해당 세션이 update 쿼리 후 commit하기 전까지 다른 세션들이 해당 row를 수정하지 못하도록 하는 기능이다.

기본 사용법

```sql

select 컬럼 from 테이블 where 조건절 for update;

```

일반적으로 select for update 구문은 트랜잭션 안에서 사용되며, 데이터베이스에서 특정 레코드를 그 레코드에 대해 배타적 잠금을 설정한다.


### 예제

먼저 auto commit이 활성화 되어 있지는 않는지 확인해보자.

```sql

SET autocommit = 0; -- auto commit 종료

```


예를 들어서 user 테이블의 이름을 수정한다고 가정해보자.

세션 1에서 user 테이블의 id가 1인 row의 데이터의 lock을 획득한다.

```sql

-- 세션 1에서 lock 획득
select * from user where id = 1 for update;

```

이때 세션 2에서 해당 데이터를 수정을 하려고 시도할 때 기다리다 에러가 발생하는 것을 확인할 수 있다.

```sql
-- 세션 2
update user set name = 'aftername' where id = 1;

-- SQL Error [1205] [40001]: Lock wait timeout exceeded; try restarting transaction

```

![sql](/public/img/240909/image.png)

세션 1에서 커밋을 한다면 세션 2에서도 데이터 수정이 가능한 모습을 볼 수 있다.

```sql
-- 세션 1

commit;
```


select for update문은 잠금을 하므로, 과도하게 사용하면 교착 상태나 성능 저하가 발생할 수 있기 때문에 필요한 경우에만 신중히 사용하는 것이 좋다.