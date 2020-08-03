# 다양한 연관관계 매핑

### 1) 일대다

#### 단방향

```java
// Team 클래스의 참조필드
@OneToMany
@JoinColumn(name="TEAM_ID")
private List<Member> members = new ArrayList<>();
```

- `팀` → `회원` 방향으로 연관관계
- 다쪽에 있는 회원에는 참조 필드가 없음
- 반대쪽 테이블에 있는 외래키를 관리하는 특이한 구조
- `@JoinColumn`을 명시해야 한다 (명시하지 않으면 조인 테이블 전략을 사용)

```java
Member member1 = new Member("member1");
Member member2 = new Member("member2");
Team team1 = new Team("team1");
team1.getMembers().add(member1);
team1.getMembers().add(member2);

em.persist(member1); // INSERT
em.persist(member2); // INSERT
em.persist(team1); // INSERT + UPDATE + UPDATE
```

- 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있기 때문에\
**추가적인 UPDATE SQL문**이 필요하게 됨
- Member 엔티티를 저장할 때는 Team에 대한 외래키가 null(빈값)으로 저장되고,\
Team 엔티티를 저장할 때 members의 참조값을 확인해서 Team에 대한 외래 키를 업데이트한다
- **일대다 단방향 매핑보다는 다대일 양방향 매핑이 권장된다**

#### 양방향

```java
// Team 클래스
@OneToMany
@JoinColumn(name="TEAM_ID")
private List<Member> members = new ArrayList<>();

// Member 클래스
@ManyToOne
@JoinColumn(name="TEAM_ID", insertable=false, updatable=false)
private Team team;
```

- `@OneToMany`는 연관관계의 주인이 될 수 없고,
다多 쪽에 외래 키가 있기 때문에 항상 `@ManyToOne`이 붙은 쪽이 주인이다
- 일대다 양방향 매핑이 완전히 불가능한 것은 아니다: 읽기 전용 매핑으로 설정
- 일대다 단방향 매핑이 갖는 단점을 그대로 갖게 되므로\
마찬가지로 **다대일 양방향 매핑 사용이 권장된다**

### 2) 일대일

- `@OneToOne`을 사용해서 객체를 매핑
- 주/대상 테이블 어느 곳이나 외래 키를 가질 수 있고
외래 키 하나만 있으면 양쪽으로 조회 가능하므로 외래 키를 누구에게 줄지를 선택해야 함
- **주 테이블에 외래 키**를 두면\
객체 참조와 비슷하게 접근할 수 있는 장점이 있다
- **대상 테이블에 외래 키**를 두면\
일대일 → 일대다로 테이블 관계가 바뀔 때 테이블 구조를 그대로 유지할 수 있는 장점이 있다
- `회원`과 `사물함`의 관계 예시) 한 회원에 하나의 사물함 할당

#### 주 테이블에서의 외래 키

```java
// Member 클래스 참조 필드
@OneToOne
@JoinColumn(name="LOCKER_ID")
private Locker locker;
```

- 일대일 단방향 매핑
- 다대일 단방향과 거의 비슷한 관계 구조를 가짐

```java
// Member 클래스는 위와 같음

// Locker 클래스
@OneToOne(mappedBy="locker")
private Member member;
```

- 일대일 양방향 매핑
- **양방향은 연관관계의 주인이 정해져야 함**
- 외래 키를 갖고 있는 Member 엔티티의 member.locker가 주인이 됨\
따라서 Locker 엔티티에서 mappedBy 속성을 이용해 주인이 아님을 명시

#### 대상 테이블에 외래 키

- **JPA는 대상 테이블에 외래 키가 있는 단방향 관계를 지원하지 않음**
- 관계 방향을 수정하거나, 양방향 관계로 만들어서 주인을 대상 테이블로 설정하는 방법밖에 없다

```java
// Member 클래스
@OneToOne(mappedBy="member")
private Locker locker;

// Locker 클래스
@OneToOne
@JoinColumn(name="MEMBER_ID")
private Member member;
```

- 대상 테이블에 외래 키가 있는 일대일 양방향
- Member 엔티티 대신 Locker 엔티티를 주인으로 지정해서\
Locker 테이블의 외래 키를 관리하도록 함

### 3) 다대다

![](https://upload.wikimedia.org/wikipedia/commons/0/02/Databases-ManyToManyWJunction.jpg)
[(참고)](https://en.wikipedia.org/wiki/Many-to-many_(data_model))

- **관계형 데이터베이스에서**는 다대다 관계로 표현할 수 없음
- 따라서 일대다, 다대일 관계로 풀어내는 **연결 테이블** _Junction table_ 을 사용\
예) 작가 ID와 책 ID를 갖는 `작가-책` 테이블을 중간에 추가
- 테이블과 달리 객체에서는 객체 2개로 다대다 관계를 만들 수 있음

#### 단방향

```java
// Member 클래스
@ManyToMany
@JoinTable(name="MEMBER_PRODUCT", joinColumns=@JoinColumn(name="MEMBER_ID"),
		inverseJoinColumns=@JoinColumn(name="PRODUCT_ID"))
private List<Product> products = new ArrayList<Product>();
```

- `@ManyToMany`와 `@JoinTable`을 사용해서\
따로 MemberProduct와 같은 엔티티 없이 연결 테이블을 바로 매핑
- **@JoinTable의 속성**\
`name` : 연결 테이블 지정\
`joinColumns` : 현재 방향(회원)과 매핑할 조인 컬럼 정보 지정\
`inverseJoinColumns` : 반대 방향(상품)과 조인할 조인 컬럼 정보 지정

```java
// Service
Product product = new Product();
product.setName("상품");
em.persist(product);

Member member = new Member();
member.setUsername("회원");
member.getProducts().add(product);
em.persist(member);
```

- INSERT문이 MEMBER, PRODUCT, MEMBER_PRODUCT 각 테이블에 실행된다
- 조회할 때도 단순하게 Member의 getProducts()를 호출하면 된다

#### 양방향

```java
// Member 클래스의 참조 필드는 그대로 (연관관계 주인)

// Product 클래스
@ManyToMany(mappedBy="products")
private List<Member> members;
```

- 역방향에도 참조 필드에 `@ManyToMany`만 저장
- 양방향 연관관계의 주인이 아닌 엔티티에 mappedBy 속성을 붙여준다

```java
// 양방향 연관관계 설정
member.getProducts().add(product);
product.getMembers().add(member);

// 연관관계 편의 메소드 정의 (Member 엔티티 내부)
public void addProduct(Product product) {
	products.add(product);
	product.getMembers().add(this);
}
member.addProduct(product); // 한번 호출로 양방향 설정
```

### 다대다 매핑의 한계

- 실무 도메인에서는 연결 테이블에 컬럼들이 단순하지 않고\
ID 뿐만 아니라 여러 칼럼들이 존재 (ex. 수량, 날짜, ...)
- 결국 연결 테이블을 매핑하는 **연결 엔티티**를 만들어서 이 엔티티에 컬럼들을 매핑해야 함\
예) `Member` - `Order(MemberProduct)` - `Product`

#### 대안 1. 복합 키 사용

- **복합 기본 키**\
연결 엔티티는 기본 키로 `MEMBER_ID`와 `PRODUCT_ID`를 가짐 → 복합 키\
이 복합 (기본)키는 별도의 식별자 클래스가 필요
- **식별자 클래스 특징**
  + `Serializable` 인터페이스 구현
  + `equals`와 `hashCode` 메소드 구현
  + 기본 생성자
  + public 클래스
  + `@IdClass`나 `@EmbeddedId`를 붙여 지정
- **식별 관계** _Identifying Relationship_ \
부모 테이블의 기본 키를 받아 자신의 기본 키이자 외래 키로 사용하는 것

#### 대안 2. 새로운 기본 키 사용

```java
// Order 클래스
@Id
@GeneratedValue
@Column(name="ORDER_ID")
private Long id;

@ManyToOne
@JoinColumn(name="MEMBER_ID")
private Member member;

@ManyToOne
@JoinColumn(name="PRODUCT_ID")
private Product product;

// Member 클래스
@OneToMany(mappedBy="member")
private List<Order> orders = new ArrayList<Order>();
```

- **비식별 관계**\
부모 테이블의 기본 키를 단순히 외래 키로만 사용하는 것
- DB에서 자동으로 생성해주는 **대리 키**를 Long 값으로 사용한다
- 간편하고, 영구적이며, 비지니스에 비의존적이라는 장점이 있다 **(추천)**
