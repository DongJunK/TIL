# Redis 기본 정리

## Redis란?

 Redis는 데이터베이스, 캐시, 메시지 브로커 및 스트리밍 엔진으로 사용되는 오픈 소스이며, In-memory 데이터 저장소이며, 1ms 미만의 응답 시간을 제공하여 다양한 분야에서 실시간 애플리케이션을 위해 초당 수백만 건의 요청을 지원할 수 있습니다.

 키-값 기반이기 때문에 키를 통해 값을 바로 가져올 수 있고, 디스크에 데이터를 쓰는 구조가 아니라 메모리에서 데이터를 처리하기 때문에 속도가 빠르다는 장점이 있습니다.

## **Redis의 특징**

- Key-Value Store: 데이터를 지칭하는 Key와 데이터를 저장하는 Value의 구조로 데이터를 저장하는 형태
- 많은 자료구조(Collection)을 제공합니다.
- Replication을 제공해서 더 서비스를 안정적으로 제공할 수 있습니다.
- Cluster 모드를 제공합니다.
- Open Source(BSD 3 License)

## **Redis data types**

Redis는 기본적으로 자료구조가 Hash Table입니다.

![https://blog.kakaocdn.net/dn/TpqYE/btrPDdKek0M/QokYV070K4aadWxC6ZKmdK/img.png](https://blog.kakaocdn.net/dn/TpqYE/btrPDdKek0M/QokYV070K4aadWxC6ZKmdK/img.png)

### Strings

- Key/Value를 사용하는 자료구조
- Key를 이용해서 Data를 저장하고 가져오게 됩니다.
- Key와 Value를 저장하는 set 명령과 Key로 Value를 가져오는 get 명령이 있습니다.
    - set <Key> <Value>
    - get <Key>
- 기본적으로 문자열을 저장하며, 직렬화된 객체, binary data, JPEG 이미지도 저장 가능합니다.
- 기본적으로 단일 Redis 문자열은 최대 512MB입니다.

### Bitmaps

- string의 변형
- bit 단위 연산이 가능합니다.
- 2^32bit까지 사용 가능합니다.
- 저장할 때, 저장 공간 절약에 큰 장점이 있습니다.

### Lists

- array 형식의 데이터 구조, 데이터를 순서대로 저장합니다.
- 추가 / 삭제 / 조회하는 것은 O(1)의 속도를 가지지만, 중간의 특정 index 값을 조회할 때는 O(N)의 속도를 가지는 단점이 있습니다.
- 즉, 중간에 추가/삭제가 느리기 때문에, head-tail에서 추가/삭제를 합니다. (push / pop 연산)
- 주로 Queue 형태의 자료구조가 필요할 때 많이 사용합니다.
    - SideKiq이나 Jesqeue라는 Redis 기반의 Queue들이 존재합니다.

### Hashes

- field-value로 구성되어 있는 전형적인 hash의 형태
- key 하위에 subkey를 이용해 추가적인 Hash Table을 제공하는 자료구조
    - Hset <key> <subkey> <value> <subkey> <value>
    - Hget <key> <subkey> <subkey> <subkey>
- 메모리가 허용하는 한, 제한 없이 field들을 넣을 수 있습니다.
- 일반적인 Key/Value 데이터를 특정군의 데이터로 묶고 싶을 때 유용합니다.

### Sets

- 중복된 데이터를 담지 않기 위해 사용하는 자료구조
- 친구리스트, 팔로워리스트 등을 저장하는데 사용할 수 있습니다.
- Spring-Security-OAuth에서 AccessToken을 저장하는 RedisTokenStore가 자료구조 Set을 사용합니다.

### Sorted Sets

- set에 score라는 필드가 추가된 데이터 형
- 데이터가 저장될 때부터 score 순으로 정렬되며 저장합니다.
- 데이터는 오름차순으로 내부 정렬됩니다.
- value는 중복 불가능, score는 중복 가능합니다.
- Score는 Double 형태이므로, 정수 값을 저장할 수 없습니다.
- 만약 score 값이 같으면 사전 순으로 정렬되어 저장됩니다.
- Skiplist 자료구조를 이용합니다.
    - Skiplist: Log(N)의 검색 속도를 가지는 리스트 자료구조

### **Redis is Single Thread**

Redis 4.0 부터는 기본적으로 4개의 쓰레드로 동작하지만 일반 명령어를 처리하는 메인쓰레드 1개와 별도의 시스템 명령을 사용하는 전용 sub thread 3개로써, 실제로 사용자가 사용하는 명령어들을 싱글쓰레드로 동작합니다.

(Redis 6.0부터 ThreadIO가 추가되어 사용자 명령에 대해서 멀티쓰레드로 처리할 수 있음, 하지만 명령어를 실행하는 코어부분은 single thread로 동작하며, I/O Socket read/write 할 때 멀티쓰레드로 동작하여 전반적인 성능이 향상됨)

- Redis는 싱글 스레드로 동작하기 때문에 long-time 명령 수행시 다른 명령어들을 처리할 수 없는 상태가 되기 때문에 성능상 문제가 됩니다.
- 주의 사용해야 할 명령어
    - keys: Redis에 존재하는 모든 key를 조회함으로써 long-time 수행을 일으킴(운영환경에서는 사용하지 말 것을 권고함)
    - seem: set 자료구조의 모든 member들을 조회함으로써 long-time 수행을 일으킴
    - flushall: 전체 데이터를 지우는 명령어로, 키 갯수에 비례한 수행시간을 갖음
    - 위 명령어들은 아래와 같이 수정하여 효율적으로 사용할 수 있음
        - keys: scan을 통해 순회 탐색으로 전체 key 조회
        - seem: sscan을 통한 순회 탐색으로 전체 member 조회

출처: The RED : 백엔드 에센셜 : 대용량 서비스를 위한 아키텍처 with Redis by 강대명