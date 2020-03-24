### MySQL 테이블 생성 및 생성 시 설정

#### Query문

```sql
CREATE TABLE [테이블명](
  컬럼1 데이터타입(자리수) OPTION(NOT NULL AUTO_INCREMENT UNIQUE) ,
  컬럼2 데이터타입(자리수) OPTION(DEFAULT '0') ,
  컬럼3 데이터타입(자리수) OPTION
  PRIMARY KEY (컬럼1)
) CHARSET=charset COLLATE=collate ENGINE=engineName ;
```



#### Charset

문자셋(character set)은 심볼(글자)과 인코딩의 묶음이고, Collation은 문자셋의 문자들을 비교하는 규칙이다.

MySQL은 전세계 모든 언어가 21bit (3바이트가 조금 안됨)에 저장되기 때문에 MySQL에서 utf8 을 3바이트 가변 자료형으로 설계하였다.  그렇기 때문에 최근에 나온 4바이트 문자열(Emoji 추가)을 utf8에 저장하면 값이 손실되는 현상이 발생한다. 

기존에 사용되던 MySQL에서 charset 은 utf8 , collation 은 utf8_general_ci 인데 이러한 대부분의 환경에서 문제를 일으키는 것이다

- utf8 : Basic Plane.

- utf8mb4 : Basic Plane + Supplementary Plane.
  - SMP (Supplementary Multilingual Plane)



- ex) latin1 (2바이트), utf8 (가변3바이트), utf8mb4 (가변4바이트)



#### Collation(정렬 방식)

Collation 은 텍스트 데이터를 정렬(ORDER BY)할 때 사용한다. 즉 text 계열 자료형에서만 사용할 수 있는 속성이다.

int 형 : 1500 < 1700  으로 명확하다.

date 형 : 2013-10-20 < 2015-12-20 으로 명확하다.

- 그렇다면 text 형에서는 어떻게 될까

  - a  와  b  중 어느 것이 더 큰가?

  - a 와 A 중 어느 것이 더 큰가?

  - a 와 á 중 어느 것이 더 큰가?



- utf8_bin (or utf8mb4_bin)
  - 바이너리 저장 값 그대로 정렬한다.
  - A - B - a - b
- utf8_general_ci (or utf8mb4_general_ci)
  - 텍스트 정렬할 때 a 다음에 b 가 나타나야 한다는 생각으로 나온 정렬방식. 일반적으로 널리 사용된다.
  - 라틴 계열 문자를 사람의 인식에 맞게 정렬한다.
  - A - a - B - b
- utf8_unicode_ci (or utf8mb4_unicode_ci)
  - 한국어, 영어, 중국어, 일본어 사용환경에서는 general_ci 와 unicode_ci 의 결과가 동일하다.
  - 더 특수한 문자의 정렬 순서가 변경된다.
  - a - á - b



utf8_general_ci는 정렬 성능을 우선시하며, 일반적인 경우에 사용하는 collation 이다. 이를 사용하면 ÀÁÅåāă 등의 문자가 없어 A 로 치환되어 비교 처리된다. 정확한 비교, 정렬 등이 필요한 경우라면 utf8_general_ci를 사용하지 않도록 한다. 물론 비교, 정렬에서 utf8_unicode_ci에 비해서 빠른 속도를 보여준다.



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



### FOREIGN KEY

```sql
CONSTRAINT FOREIGN KEY (colum) REFERENCES table_name(colum)
ON DELETE referential_action
ON UPDATE referential_action
```



InnoDB는 부모 테이블 (parent table) 안에 매칭되는 후보 키 값이 존재하지 않을 경우에는 차일드 테이블에서 foreign 키 값을 생성하고자 시도하는 모든  INSERT 또는 UPDATE 연산을 거부한다. 차일드 테이블에 매칭되는 열을 가지고 있는 부보 테이블에서 후보 키 값을 업데이트 또는 삭제하고자 시도하는 UPDATE 또는 DELETE 연산을 실행하는 InnoDB의 동작은 FOREIGN 키 구문의 ON UPDATE 및 ON DELETE 서브 구문을 사용해서 지정되는 **referential action** 에 달려 있게 된다. 사용자가 부모 테이블에서 열을 업데이트 또는 삭제하고자 시도하고, 차일드 테이블에 매칭되는 열이 하나 또는 그 이상이 있는 경우, InnoDB는 실행되는 동작에 관련하여 5가지의 옵션을 지원한다

- Referential Action
  - **CASCADE**: 부모 테이블에서 열을 삭제 또는 업데이트하고 차일드 테이블에서 매칭 열을 자동으로 삭제 또는 업데이트 한다. **ON DELETE CASCADE** 및 **ON UPDATE CASCADE**는 모두 지원된다. 두 테이블간에는, 부모 테이블 또는 차일드 테이블에 있는 동일한 컬럼에서 작용하는 **ON UPDATE CASCADE** 구문을 정의하지 말아야 한다.
  - **SET NULL**: 부모 테이블에서 열을 삭제 또는 업데이트하고 foreign 키 컬럼 또는 차일드 테이블에 있는 컬럼을 **NULL**로 설정한다. 이것은 foreign 키 컬럼이 **NOT NULL** 수식어를 가지고 있는 않을 경우에만 유효하다. **ON DELETE SET NULL** 및 **ON UPDATE SET NULL** 구문은 모두 지원된다.
  - **NO ACTION**: 표준 SQL에서는, **NO ACTION** 은 말 그대로 프라이머리 키 값을 삭제 또는 업데이트하려는 시도가 레퍼런스 테이블에 관련된 foreign 키 값이 존재할 경우에는 허용되지 않는다는 것을 의미한다. **InnoDB**는 부모 테이블에 대한 삭제 또는 업데이트 연산을 거부한다.
  - **RESTRICT**: 부모 테이블에 대한 업데이트 또는 삭제 연산을 거부한다 (Reject). **NO ACTION** 및 **RESTRICT**는 **ON DELETE** 또는 **ON UPDATE** 구문을 생략한 것과 같은 것이다. (어떤 데이터베이스 시스템은 검사 연기 (deferred check)를 가지고 있는데, **NO ACTION**이 검사 연기이다. MySQL의 경우, FOREIGN 키 제한은 즉시 검사를 진행하기 때문에, **NO ACTION** 및 **RESTRICT**는 동일한 것이 된다.)
  - **SET DEFAULT**: 이 동작은 파서에 의해 인식되지만, **InnoDB**는 **ON DELETE SET DEFAULT** 또는 **ON UPDATE SET DEFAULT** 구문을 가지고 있는 테이블 정의문을 거부한다.



### Query 작성과 Workbench 생성 Query

- UNIQUE 옵션은 UNIQUE INDEX로 변경
- FOREIGN KEY는 INDEX(ASC) 설정됨
- FOREIGN KEY의 ON DELETE와 UPDATE의 referential action은 NO ACTION으로 설정됨