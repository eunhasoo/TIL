# JPQL

### JPA를 사용한 검색 방법들

1. **JPQL** _Java Persistence Query Language_
2.  Criteria 쿼리: JPQL을 편하게 작성하도록 돕는 API, 빌더 클래스 모음
3. 네이티브 SQL 쿼리: JPQL 대신 직접 SQL을 사용하도록 지원
4. QueryDSL: Criteria 쿼리와 비슷하게 JPQL 작성이 편리하도록 돕는 빌더 클래스 모음
5. JDBC 직접 사용
6. MyBatis 같은 SQL Mapper 프레임워크 사용

### 개념과 특징

```java
// SQL) SELECT member.id as id, member.name as name FROM Member member WHERE member.name='kim'
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
```

- JPQL은 `엔티티 객체`를 조회하는 객체지향 쿼리
- 데이터베이스 방언 _Dialect_ 변경만으로 JPQL 수정없이 데이터베이스를 변경할 수 있음
- SQL보다 간결하고 다양한 기능이 제공됨

#### SQL과 JPQL

- SQL
  + 데이터베이스 테이블을 대상으로 탐색
  + 특정 DBMS에 의존
- JPQL
  + 객체를 대상으로 탐색
  + SQL을 추상화하여 특정 DBMS에 의존하지 않음

### 기본 문법

- SELECT, UPDATE, DELETE문 제공\
(INSERT는 persist() 메소드 사용)
- 전체 구조(순서)는 SQL과 동일
- **엔티티와 속성에 대소문자 구분**\
SELECT 같은 JPQL 키워드는 구분하지 않음
- 클래스 명이 아니라 **엔티티 명**을 사용
- **별칭 필수 사용**\
예) SELECT `m`.username Member `m`

### TypeQuery와 Query

```java
List<Member> list = em.createQuery("SELECT m FROM Member m where m.username = :username", Member.class)
				.setParameter("username", "kim") // 파라미터 바인딩
				.getResultList(); // 메소드 체이닝 사용
```

- 반환 타입이 명확하거나 한 가지면 `TypeQuery`,\
반환 타입을 명확히 지정할 수 없거나 여러 대상인 경우 `Query`를 사용한다
- Query 객체를 사용하면 조회 대상이 하나일 때 `Object`, 여러 개일 때 `Object[]`를 반환하고\
사용하기 위해서는 타입을 변환해줘야 한다
- JPQL API는 대부분 `메소드 체이닝` 방식으로 설계되어 있다
- `:파라미터명`을 사용해 이름 기준 _Named_ 파라미터를 정의하고\
setParameter() 메소드로 파라미터를 바인딩할 수 있다
- SQL 인젝션 공격을 피하기 위해서라도 쿼리를 작성할 때는\
파라미터 바인딩 방식을 사용하는 것이 좋다

### 프로젝션

- SELECT 절에 조회할 대상을 지정하는 것을 의미
- 엔티티, 엠비디드 타입, 스칼라 타입 지정 가능
- 엔티티 프로젝션으로 조회된 엔티티는 영속성 컨텍스트에서 관리
- 임베디드 타입 프로젝션은 시작점이 될 수 없는 제약이 있음
  + 틀린 예) SELECT a FROM address a
  + 옳은 예) SELECT o.address FROM Order o

```java
// 프로젝션으로 여러 값을 선택하면 `Query` 객체를 사용해야 함
Query query = em.createQuery("SELECT m.username, m.age FROM Member m");
List<Object[]> resultList = query.getResultList();
for (Object[] row : resultList) {
	String username = (String) row[0];
	Integer age = (Integer) row[1];
	// ....
}

// NEW 명령어로 간단하게 객체 변환
String jpql = "SELECT new com.eunhasoo.dto.UserDTO(m.username, m.age) FROM Member m";
TypedQuery<UserDTO> query = em.createQuery(jpql, UserDTO.class);
List<UserDTO> resultList = query.getResultList();
```

### JPQL 조인

#### 내부 조인

```sql
SELECT m FROM Member m
[INNER] JOIN m.team t
WHERE t.name = :teamName
```

- 내부 조인은 INNER JOIN을 사용하고\
INNER는 생략할 수 있다
- `:teamName`에 파라미터를 바인딩하여 필터링된 조회 결과를 얻을 수 있다
- **연관 필드를 사용하여 조인**하기 때문에
INNER JOIN `Team`과 같이 SQL 조인처럼 사용하면 오류가 발생한다

```java
String query = "SELECT m, t FROM Member m JOIN m.team t";
List<Object[]> result = em.createQuery(query).getResultList; // 타입이 다르므로 TypeQuery 객체 사용불가
for (Object[] row : result) {
	// ...
}
```

#### 외부 조인

```sql
SELECT m FROM Member m
LEFT [OUTER] JOIN m.team t
```

- SQL의 외부 조인과 같고\
OUTER는 생략 가능하다
- <참고> [OUTER 조인](https://www.zentut.com/sql-tutorial/sql-outer-join/)

#### 컬렉션 조인

```sql
SELECT t, m FROM Team t
LEFT JOIN t.members m
```

- 다대일 조인은 단일 값 연관 필드 사용
- 일대다 조인은 컬렉션 값 연관 필드 사용

#### 페치 조인

```sql
SELECT m FROM Member m
JOIN FETCH m.team
```

- 연관된 엔티티나 컬렉션을 **한번에 같이** 조회하는 기능
- `JOIN FATCH` 명령어로 사용
- `select m` 으로 회원 엔티티만 선택해도\
회원과 연관된 팀 객체도 함께 조회된다
- team 필드를 지연 로딩으로 설정했더라도\
함께 조회되기 때문에 지연 로딩이 일어나지 않는다

#### 컬렉션 페치 조인

```sql
SELECT t FROM Team t
JOIN FETCH t.members
```

- 팀을 조회하면서 연관된 회원을 같이 조회 (일대다)
- 일대다 조인은 다른 연관관계와 다르게\
결과로 나오는 값의 개수가 증가할 수 있다

#### 페치 조인과 일반 조인의 차이

```sql
-- 일반 조인 JPQL
SELECT t FROM Team t
JOIN t.members m
WHERE t.name='팀A'

-- 실행 결과 SQL
SELECT T.* FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
WHERE T.NAME='팀A'
```

- Team만 조회되고 조인했던 Members는 조회가 되지 않음
- 연관관계를 고려하지 않기 때문에 SELECT 절에 지정한 엔티티만 조회했음

```sql
-- 컬렉션 패치 조인 JPQL
SELECT t FROM Team t
JOIN FETCH t.members
WHERE t.name='팀A'

-- 실행 결과 SQL
SELECT T.*, M.* FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
WHERE T.NAME='팀A'
```

- Team과 함께 Members도 조회되었음
- 연관 엔티티들을 함께 조회하면서 SQL 호출 횟수를 줄일 수 있음
- FetchType으로 엔티티에 적용하는 `글로벌 로딩 전략`보다\
우선시 되기 때문에 페치 조인을 사용하면 페치 조인을 적용해 즉시 함께 조회
- 글로벌 로딩 전략을 즉시 로딩으로 설정하면 애플리케이션 전체에서\
항상 즉시 로딩이 일어나기 때문에 성능에 악영향을 줄 수 있음\
→ 최적화를 위해 되도록 페치 조인을 쓰자

### 경로 표현식

- `.`을 찍어서 객체 그래프를 탐색하는 것
- `m.team`, `t.name`과 같이 사용됨
- 상태, 연관 필드로 나뉘어짐

#### 상태 필드

- 상태 _state_ 필드는 단순히 값을 저장하기 위한 필드
- 경로 탐색의 끝으로, 더 이상 탐색할 수 없음

#### 연관 필드

- 연관 _association_ 필드는 연관관계를 위한 필드로 임베디드 타입이 포함됨
- 단일 값 연관 필드: `@ManyToOne`, `@OneToOne` ← 대상이 엔티티\
묵시적으로 내부 조인이 일어나고, 계속 탐색 가능
- 컬렉션 값 연관 필드: `@ManyToMany`, `@OneToMany` ← 대상이 컬렉션\
묵시적으로 내부 조인이 일어나고, \
더 이상 탐색할 수 없지만 조인으로 별칭을 얻으면 해당 별칭으로 탐색 가능

#### 묵시적/명시적 조인

- 명시적 조인: JOIN 키워드를 직접 적어주는 것\
(예) `SELECT m FROM Member m `**`JOIN m.team t`**
- 묵시적 조인: 경로 표현식에 의해 묵시적으로 조인이 일어나는 것 **(항상 내부 조인)**
(예) `SELECT m.team FROM `**`Member m`**
- 성능이 중요하면 분석하기 쉽도록 명시적으로 작성하는 것이 좋다

### 서브 쿼리

- WHERE, HAVING 절에서만 사용 가능
- EXIST / ALL / ANY / SOME / IN 함수 사용 가능

### 컬렉션 조건식

```sql
-- 주문이 하나 이상 있는 회원 조회
SELECT m FROM Member m
WHERE m.orders is not empty

-- 파라미터에 정의된 회원이 있는 팀 조회
SELECT t FROM Team t
WHERE :memberParam member of t.members
```

- 컬렉션은 컬렉션 식 이외에 다른 식을 사용할 수 없다
- 컬렉션에 IS NULL을 사용하면 오류 발생

### 엔티티 직접 사용

```sql
-- 둘다 결과값이 같다
SELECT COUNT(m.id) FROM Member m
SELECT COUNT(m) FROM Member m

-- 외래 키 사용
SELECT m.* FROM Member m
WHERE m.team=:team
```

- JPQL에서 엔티티 객체를 직접 사용하면\
**해당 엔티티의 기본 키 값**을 사용한다
- 마찬가지로 외래 키로 매핑된 엔티티를 사용하면\
해당 엔티티의 식별자 값이 적용된다
