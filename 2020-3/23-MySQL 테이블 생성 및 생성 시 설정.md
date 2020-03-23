### MySQL 테이블 생성 및 생성 시 설정

#### Query문

```sql
CREATE TABLE [테이블명]

(

  컬럼1 데이터타입(자리수) OPTION ,

  컬럼2 데이터타입(자리수) OPTION ,

  컬럼3 데이터타입(자리수) OPTION

) ENGINE=engineName;
```



#### Storage Engine

MySQL은 크게 2가지 구조로 되어있다.

1. 서버 엔진 - 클라이언트(또는 사용자)가 Query를 요청했을때 Query Parsing과 스토리지 엔진에 데이터를 요청하는 작업을 수행
2. 스토리지 엔진 - 물리적 저장장치에서 데이터를 읽어오는 역할을 담당



스토리지 엔진은 데이터를 직접적으로 다루는 역할을 하므로 엔진의 종류마다 동작 원리가 다르고, 따라서 트렌젝션, 성능과 같은 주요 이슈에도 밀접하게 연관되어있다.



MySQL의 스토리지 엔진은 Plug In 방식이며, 기본적으로 8가지 스토리지 엔진이 탑재되어 있다.

```sql
SHOW ENGINES;
```

위의 명령어로 탑재된 스토리지 엔진을 확인할 수 있다.



가장 많이 쓰이는 엔진으로는 MyISAM, InnoDB, Archive가 있다.



- **InnoDB** : 따로 스토리지 엔진을 명시하지 않으면 default 로 설정되는 스토리지 엔진이다. InnoDB는 transaction-safe 하며, 커밋과 롤백, 그리고 데이터 복구 기능을 제공하므로 데이터를 효과적으로 보호 할 수 있다.

  InnoDB는 기본적으로 row-level locking 제공하며, 또한 데이터를 clustered index에 저장하여 PK 기반의 query의 I/O 비용을 줄인다. 또한 FK 제약을 제공하여 데이터 무결성을 보장한다.

  

- **MyISAM** : 트랜잭션을 지원하지 않고 table-level locking을 제공한다. 따라서 multi-thread 환경에서 성능이 저하 될 수 있다. 특정 세션이 테이블을 변경하는 동안 테이블 단위로 lock이 잡히기 때문이다.

  텍스트 전문 검색(Fulltext Searching)과 지리정보 처리 기능도 지원되는데, 이를 사용할 시에는 파티셔닝을 사용할 수 없다는 단점이 있다.

  

- **Archive** : '로그 수집'에 적합한 엔진이다. 데이터가 메모리상에서 압축되고 압축된 상태로 디스크에 저장이 되기 때문에 row-level locking이 가능하다.

  다만, 한번 INSERT된 데이터는 UPDATE, DELETE를 사용할 수 없으며 인덱스를 지원하지 않는다. 따라서 거의 가공하지 않을 원시 로그 데이터를 관리하는데에 효율적일 수 있고, 테이블 파티셔닝도 지원한다. 다만 트랜잭션은 지원하지 않는다.



[MySQL 공식 Document - storage-engines](https://dev.mysql.com/doc/refman/5.7/en/storage-engines.html) 문서를 확인하면 다른 스토리지 엔진에 대한 내용도 확인할 수 있다.

