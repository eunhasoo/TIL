# 엔티티 매핑

### 1) 엔티티-테이블 매핑

#### @Entity

- JPA를 이용해 테이블과 매핑할 클래스에 필수로 붙이는 어노테이션
- 이 어노테이션을 붙인 클래스는 JPA가 관리한다
- **주의사항**\
  (1) 기본 생성자 필수\
  (2) final 클래스, enum, interface, inner 클래스에 사용 불가\
  (3) 저장할 필드에 final 사용 불가

#### @Table

- 엔티티와 매핑할 테이블을 지정
- 생략시 매핑한 엔티티 이름을 테이블 이름으로 사용

### 2) 기본 키 매핑

#### JPA가 제공하는 기본 키 생성 전략

- **직접 할당**: 기본 키를 애플리케이션에서 직접 할당. `@Id`만 사용
- **자동 생성** : 대리 키 사용 방식. `@Id`와 `@GeneratedValue` 사용
  + `IDENTITY`: 기본키 생성을 데이터베이스에 위임
  + `SEQUENCE`: 데이터베이스 시퀀스를 사용해서 할당
  + `TABLE`: 키 생성 테이블 사용를 사용해서 할당

#### 직접 할당

```java
// Entity
@Id
@Column(name="id")
private String id;

// Service
Board board = new Board();
board.setId("00001"); // 직접 기본 키를 할당한다
em.persist(board);
```

- 엔티티 저장 전에 애플리케이션에서 기본 키를 직접 할당하는 방식
- `@Id`로 매핑될 수 있는 자바 타입\
기본형, Wrapper형, String, util.Date, sql.Date, math.BigDecimal, math.BigInteger

#### IDENTITY 전략

```java
// Entity
@Id
@GeneratedValue(strategy=GenerationType.IDENTITY)
private Long id;

// Service
Board board = new Board();
em.persist(board);
```

- 주로 MySQL, PostgreSQL, SQL, SQL Servce, DB2에서 사용
- DB에 값(엔티티)을 저장하고 나서야 기본 키 값을 구할 수 있음
- @GeneratedValue의 strategy 속성을 `GeneratedType.IDENTITY`으로 지정
- JPA가 기본 키 값을 얻어오기 위해 추가적으로 조회
- `Statement.getGeneratedKeys()`를 사용하면 데이터 저장과 동시에 생성된 기본 키값을 얻을 수 있음
- persist() 호출 즉시 INSERT 쿼리가 전달되어야 하므로 _트랜잭션을 지원하는 쓰기 지연이 동작하지 않음_

#### SEQUENCE 전략

```java
// SQL 시퀀스 생성
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;

// Entity
@Entity
@SequenceGenerator(
	name="BOARD_SEQ_GENERATOR",
	sequenceName="BOARD_SEQ",
	initialValue=1, allocationSize=1)
public class Board {
	@Id
	@GeneratedValue(strategy=GenerationType.SEQUENCE,
			generator="BOARD_SEQ_GENERATOR")
	private Long id;
	...
}

// Service
Board board = new Board();
em.persist(board);
```

- **시퀀스**: 유일한 값을 순서대로 생성하는 특별한 데이터베이스 Object
- 시퀀스를 지원하는 Oracle, PostgreSQL, DB2, H2 데이터베이스에서 사용 가능
- `@SequenceGenerator`: 시퀀스 생성기를 등록
- `sequenceName`: 실제 데이터베이스 시퀀스와 매핑
- strategy를 `GenerationType.SEQUENCE`로 설정한 뒤 `generator`에 위에 등록한 생성기를 선택
- persist()를 호출할 때 먼저 시퀀스를 사용해 **식별자를 조회**하고,\
조회한 식별자를 엔티티에 할당한 뒤 엔티티를 영속성 컨텍스트에 저장하고\
트랜잭션이 커밋되면 (flush) 엔티티를 **데이터베이스에 저장**하는 과정을 거친다.
- 식별자 조회, DB 저장 총 2번 DB와 통신이 필요하다

#### TABLE 전략

```java
@Entity
@TableGenerator(
	name="BOARD_SEQ_GENERATOR",
	table="MY_SEQUENCES", // 테이블 이름
	pkColumnValue="BOARD_SEQ", allocationSize=1)
public class Board {
	@Id
	@GeneratedValue(strategy=GenerationType.TABLE,
	generator="BOARD_SEQ_GENERATOR")
	private Long id;
	...
}

// Service
Board board = new Board();
em.persist(board);
```

- 키 생성 전용 테이블을 하나 만든 뒤 이름과 값으로 사용할 컬럼을 생성하여\
데이터베이스 시퀀스를 mocking 하는 전략
- 테이블만을 사용하므로 모든 데이터베이스에서 사용할 수 있다
- `@TableGenerator`: 테이블 키 생성기를 등록하고, 키 생성용 테이블과 매핑
- `GenerationType.TABLE`: 테이블 전략 사용을 명시
- `generator`: 만들었던 테이블 키 생성기를 지정
- `pkColumnValue`: 키로 사용할 값 이름을 지정
- SEQUENCE 전략과 내부 동작 방식이 같다

#### AUTO 전략

```java
@Id
@GeneratedValue(strategy=GenerationType.AUTO) // strategy 속성 생략 가능
private Long id;
```

- 선택된 데이터베이스 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택
- 데이트베이스를 변경해도 코드 수정이 필요 없다는 장점이 있다
- SEQUENCE나 TABLE 전략이 선택될 경우 시퀀스나 키 생성용 테이블을 미리 만들어두어야 함

### <권장되는 식별자 선택 전략>

#### 데이터베이스 기본 키 조건

1. null 값을 비허용하고
2.  변하지 않는
3.  유일한 값

#### 테이블 기본 키 선택 전략
- 자연 _natural_ 키: 비즈니스에 _의미가 있는_ 키
- 대리 _surrogate_ 키: 비즈니스와 _관련 없는_ 임의로 만들어진, 대체 키 **(권장)**