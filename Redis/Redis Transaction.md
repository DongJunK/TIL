## Transaction
트랜잭션이란, 트랜잭션은 데이터베이스에서 데이터를 검색하거나 업데이트를 위해 독립적으로 실행되는 논리 단위

## Redis Transaction
Redis의 트랜잭션은 SQL 데이터베이스의 트랜잭션과 다릅니다.
Redis의 트랜잭션은 MULTI 및 EXEC 사이에 배치된 명령 블록으로 구성됩니다.

### 특징
- 트랜잭션의 모든 명령은 직렬화되고 순차적으로 실행됩니다.
- Redis 트랜잭션이 실행되는 동안 다른 클라이언트가 보낸 요청은 처리되지 않습니다.
    - 명령이 격리된 단일 작업으로 실행
- 설정에 따라 트랜잭션도 Persistence를 위해 디스크에 저장할 수 있습니다.
- Redis는 롤백을 지원하면 Redis의 단순성과 성능에 상당한 영향을 미치기 때문에 트랜잭션 롤백을 지원하지 않습니다.
    - 트랜잭션 내부에서 에러가 발생하더라도 전체 트랜잭션은 롤백되지 않으며, 에러가 발생하지 않은 명령어는 정상적으로 실행됩니다.

## Transaction 명령어
### MULTI
- Redis의 트랜잭션을 시작하는 명령어
- MULTI 명령어를 통해 트랜잭션이 실행되면 이후 커맨드는 Queue에 쌓입니다.
### EXEC
- 정상적으로 처리되며 Queue에 쌓여있는 명령어를 일괄적으로 실행하는 명령어
- EXEC 명령어를 호출하기 전에 서버에 대한 연결이 끊기면 Queue에 있던 명령은 수행되지 않습니다.
### DISCARD
- Queue에 쌓여있는 명령어를 일괄적으로 폐기하는 명령어
### WATCH
- Redis에서 Lock을 담당하는 명령어
- WATCH 명령어를 사용하면 UNWATCH 되기 전에는 한 번의 EXEC 또는 Transaction이 아닌 명령어만 허용합니다.
- Optimistic Lock 기반
### Optimistic locking using check-and-set
```
val = GET mykey
val = val + 1
SET mykey $val
```

두 클라이언트에 의해 mykey 값을 동시에 증가시키려고 하면 RACE CONDITION이 발생합니다.

예를 들어, val의 값이 10일 때, 두 클라이언트에 의해 해당 로직이 실행된다면 12가 아닌 11이 됩니다.

WATCH를 통해 해당 문제를 해결할 수 있습니다.
```
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
UNWATCH mykey
```
참고로, 보통 여러 클라이언트가 서로 다른 키에 액세스하기 때문에 키에 대한 충돌이 발생할 가능성이 낮습니다.

참고
https://redis.io/docs/manual/transactions/
https://www.techopedia.com/definition/16455/transaction-databases
https://stackexchange.github.io/StackExchange.Redis/Transactions.html