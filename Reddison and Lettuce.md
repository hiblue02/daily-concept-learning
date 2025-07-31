
<table>
  <tr>
    <td valign="top" width="50%">

## 📌 Reddison
https://redisson.pro/docs/
### Java 기반 고수준 Redis Client
- Redis의 String, Hash, Set, List 등의 자료구조를 RBucket, RMap, RSet, RList 등의 추상화된 자바 객체로 지원한다.
- 객체 직렬/역직렬화를 자동으로 처리하고, Java Collection과 유사한 구성의 인터페이스를 제공한다. 
- 분산락, 로컬 캐시 등 레디스 기능 외의 기능을 제공한다.   
```java
RMap<String, Integer> map = redisson.getMap("scoreMap");
map.put("user1", 100);
map.put("user2", 95);
```

### Netty
Redis와 연결(통신)에 Netty를 사용한다. 
> Netty는 비동기 이벤트 기반 네트워크 프레임워크이다. 논블로킹과 비동기방식을 지원한다.
> 1) 클라이언트와 쓰레드 연결 후 요청이 오지 않으면 대기한다. (다른  작업을 수행한다.)
> 2) 요청이 생기면 O/S가 이벤트로 쓰레드에 알려준다.
> 3) 쓰레드는 작업결과를 비동기로 클라이언트에 전송한다.

### 고급기능 
- 자료구조        | RMap, RSet, RList, RBucket 등         
- 동시성         | 분산 락, Semaphore(동시접근 수량제어), CountDownLatch(작업완료대기)              
- 실행기         | 분산 Executor, Scheduler, Remote Service ❓뭘까..
- 트랜잭션        | 트랜잭션 지원                           
- 캐시           | 로컬 캐시, 클라이언트 사이드 캐시, TTL               
- 네트워크/클러스터 | 자동 재시도, 장애 감지, 리디렉션
                  

> 분산락 RLock
> 1) set {key} {client_id} NX PX 3000 형태로 락을 획득한다. (이미 값이 있으면 nil을 반환해 락 획득에 실패한다.)
> 2) 거래 종료 후 락을 해제한다. 해제 전, client_id로 락의 소유자가 보낸 요청인지 확인한다. (원자성 보장을 위해 루아 스크립트로 처리한다.)
> RedLock
> 1) 여러 노드에서 락 획득을 시도하고 n개 이상 성공하면 락을 획득한다.   

### 센티널 모드 
- polling 방식으로 센티널 노드의 상태변경을 감지한다. https://redisson.pro/docs/configuration/#sentinel-settings
- 상태변경을 감지하면 센티널에 질의해 새로운 마스터노드의 정보를 받아온다. 새로운 마스터 노드로 커넥션을 바꾼다. 

### 클러스터 모드
1) 연결 할 때 `CLUSTER SLOTS`으로 해시슬롯 정보를 캐싱, 리디렉션
2) 해시슬롯에 따라 명령어를 라우팅한다. (다중 Key Command, 전체 노드 Select Command)

### 명령어 재실행 / 노드 재연결
https://redisson.pro/docs/fault-tolerance-and-recovery/
1) 연결 장애가 발생한 경우 설정에 따라 명령어를 재시도 한다. (retryDelay, retryAttempts)
2) 연결 장애가 정해진 시간 후에도 복구되지 않으면 복제 노드도 실패한 것으로 본다. 설정된 시간(3000ms) 후에 실패한 노드에 연결을 시도한다.   
</td>
    <td valign="top" width="50%">

    
## 📌 Lettuce
### Java 기본 저수준 Redis Client
- Redis 명령어를 그대로 사용한다.
- 반환 값은 기본형이며, 객체 직렬/역직렬화를 제공하지 않는다. (6.5버전 부터는 JsonObject를 지원한다.)
```java
RedisClient client = RedisClient.create("redis://localhost:6379");
StatefulRedisConnection<String, String> connection = client.connect();
RedisCommands<String, String> sync = connection.sync();

// Hash에 필드 추가
sync.hset("user:1001", "name", "Alice");
sync.hset("user:1001", "age", "30");

// 특정 필드 가져오기
String name = sync.hget("user:1001", "name");
System.out.println("Name: " + name);

// 모든 필드 가져오기
Map<String, String> userMap = sync.hgetall("user:1001");
userMap.forEach((k, v) -> System.out.println(k + ": " + v));

connection.close();
client.shutdown();
```

### Netty
- lettuce도 reddision처럼 Netty를 사용한다.
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

### 센티널 모드 
- 센티널 노드의 상태변경 이벤트(pub/sub)을 수신 받는다. https://redis.github.io/lettuce/ha-sharding/
- 센티널 노드에게 받은 정보를 가지고 새로운 마스터 노드로 연결을 바꾼다.

### 클러스터 모드
https://redis.github.io/lettuce/ha-sharding/#redis-cluster
1) 연결 할 때 `CLUSTER SLOTS`으로 해시슬롯 정보를 캐싱한다. 
2) 해시슬롯에 따라 명령어를 라우팅한다. (다중 Key Command, 전체 노드 Select Command)
3) MOVE/ASK 응답 혹은 연결이 끊기면 토폴로지 정보를 갱신하거나, 주시적으로 토폴로지 정보를 갱신한다. (기본값은 OFF, 활성화해줘야 함)

### 명령어 재실행 / 노드 재연결
https://redis.github.io/lettuce/advanced-usage/#message-ordering
1) at-least-once가 기본값으로 연결 장애가 복구되면 버퍼에 저장되어 있던 명령어가 실행된다.
2) ConnectionWatchDog ❓자료 못찾겠다. https://github.com/redis/lettuce/blob/371beb09547e85f8e981c1c4ebb70c3e3179d3d2/src/main/java/io/lettuce/core/protocol/ConnectionWatchdog.java#L62
    </td>
  </tr>
</table>
