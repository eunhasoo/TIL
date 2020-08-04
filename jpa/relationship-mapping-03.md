# 고급 매핑

### 1) 상속 관계 매핑

- 데이터베이스의 슈퍼타입 서브타입 관계 모델링 기법과 유사
- 조인 전략, 단일 테이블 전략, 구현 클래스당 테이블 전략으로 나뉨

#### 조인 전략

- 각 엔티티들을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서\
기본 키 + 외래 키로 사용하는 전략
- 타입을 구분할 수 있는 컬럼을 따로 추가해야 함
- 부모 클래스에 `@Inheritance`을 붙이고 전략을 지정, `@DiscriminatorColumn`으로 구분 컬럼을 지정
- 자식 클래스에 `@DiscriminatorValue`로 구분 컬럼에 입력할 값을 지정
- `@PrimaryKeyJoinColumn`을 사용해 name 속성에 자식 테이블의 기본 키 컬럼명을 변경할 수 있다

```java
// Item 클래스 (부모)
@Entity
@Inheritance(strategy = InheritanceType.JOINED) // 조인 전략을 사용한 부모 클래스를 명시
@DiscriminatorColumn(name = "DTYPE") // 구분 컬럼 지정. 이 컬럼으로 저장된 자식 테이블을 구분 (기본이 DTYPE)
public abstract class Item { ... }

// Movie 클래스 (자식)
@Entity
@DiscriminatorValue("M") // 엔티티를 저장할 때 구분 컬럼에 입력할 값
public class Movie extends Item { ... }
```
- **장점**
  + 테이블이 정규화됨
  + 외래 키 참조 무결성 제약조건 활용 가능
  + 효율적인 저장 공간
- **단점**
  + 조회시 조인이 많이 사용되므로 성능 저하
  + 복잡한 조회 쿼리
  + 데이터 등록시 INSERT SQL이 두번 실행

#### 단일 테이블 전략

- 테이블을 하나만 사용
- 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분
- 조회시 조인을 사용하지 않으므로 가장 빠름
- 매핑할 컬럼들은 모두 `null 허용` 이어야 함
- 테이블 하나로 모든 것을 통합하므로 구분 컬럼 사용이 필수적

```java
// Item 클래스 (부모)
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE) // 단일 테이블 전략을 사용한 부모 클래스를 명시
@DiscriminatorColumn // 구분 컬럼 지정. 기본이 DTYPE이므로 생략
public abstract class Item { ... }

// Movie 클래스 (자식)
@Entity
@DiscriminatorValue("M") // 비지정시 기본으로 엔티티 이름 사용 
public class Movie extends Item { ... }
```

- **장점**
  + 빠른 조회 성능
  + 단순한 조회 쿼리
- **단점**
  + 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 함
  + 단일 테이블에 모두 저장되므로 테이블이 커질 수 있고 그에 따른 성능 저하 우려

#### 구현 클래스당 테이블 전략

- 자식 엔티티마다 테이블을 만들어서\
각각에 필요한 컬럼이 모두 존재
- 구분 컬럼을 사용하지 않음
- 일반적으로 비추천되는 전략

```java
// Item 클래스 (부모)
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) // 구현 클래스당 테이블 전략 사용
public abstract class Item { ... }

// Movie 클래스 (자식)
@Entity
public class Movie extends Item { ... }
```

- **장점**
  + 서브 타입 구분 처리에 효과적
  + not null 제약 조건 사용 가능
- **단점**
  + 여러 자식 테이블을 함께 조회시 성능 저하 (UNION 사용)
  + 자식 테이블을 통합해 쿼리하기 어려움

### 2) 매핑 정보만 상속받기

- 부모 클래스는 테이블과 매핑하지 않고\
상속받을 자식 클래스에게 매핑 정보만 제공하고자 할 때 사용
- `@MappedSuperclass` : 추상 클래스와 비슷하지만 실제로 테이블과 매핑되지는 않고\
매핑 정보를 상속할 목적으로만 사용되는 어노테이션

```java
@MappedSuperclass // 엔티티가 아니므로 @Entity가 없다
public abstract class BaseEntity {
	@Id
	@GeneratedValue
	private Long id;
	private String name;	
	...
}

@Entity
@AttributeOverride(name="id", column=@Column(name="MEMBER_ID") // ID 컬럼만 재정의
public class Member extends BaseEntity {
	// id, name 필드를 상속 받음
	private String email;
	...
}

@Entity
@AttributeOverrides({@AttributeOverride(name="id", column=@Column(name="SELLER_ID")),
		@AttributeOverride(name="name", column=@Column(name="SELLER_NAME"))
}) // 둘 이상의 컬럼 재정의
public class Seller extends BaseEntity { ... }
```

- BaseEntity에 객체들이 주로 사용하는 공통 매핑 정보를 정의하고\
자식 엔티티들은 상속을 통해 매핑 정보를 물려 받음
- BaseEntity는 테이블과 매핑이 필요없고 물려줄 매핑 정보만 정의하면 됨
- 자식 클래스에서 부모로부터 상속받은 매핑 정보를 재정의하려면\
`@AttributeOverride`(하나) 혹은 `@AttributeOverrides`(둘 이상)를 사용
- 연관관계 재정의는 `@AssociationOverride`나 `@AssociationOverrides` 사용
- 엔티티가 아니기 때문에 find() 메소드나 JPQL로 참조될 수 없음
- 직접 생성하여 사용할 일이 없으므로 **추상 클래스**로 만드는 것이 권장
- 등록일/수정일, 등록자/수정자 등과 같은 공통 속성을 효과적으로 관리 가능

### 3) 복합 키와 식별 관계 매핑

- **식별 관계**: 부모 테이블의 기본 키를 받아 자식 테이블의 `기본 키` + `외래 키`로 사용
- **비식별 관계**: 부모 테이블의 기본 키를 받아 자식 테이블의 `외래 키`로만 사용
  + 필수적 비식별 관계: 외래 키에 NULL 비허용. 연관관계 필수적
  + 선택적 비식별 관계: 외래 키에 NULL 허용. 연관관계 선택적

### 복합키 비식별 관계 매핑

- 둘 이상의 컬럼으로 기본 키를 사용하는 복합 키는\
별도의 식별자 클래스를 만들어 사용해야 함
- 식별자 클래스 내부에는 equals와 hashCode 메소드를 구현해야 함

#### @IdClass 사용하기

```java
// Parent 클래스
@Entity
@IdClass(ParentId.class) // 식별자 클래스 지정
public class Parent {
	@Id @Column(name="PARENT_ID1")
	private String id1;

	@Id @Column(name="PARENT_ID2")
	private String id2;
}

// ParentId (식별자 클래스)
public class ParentId implements Serializable { // Serializable를 구현한 public 클래스
	// 엔티티에서 사용하는 속성명과 일치시킴
	private String id1;
	private String id2;

	public ParentId() { } // 기본 생성자

	// equals, hashCode 구현
}

// Child 클래스
@Entity
public class Child {
	@Id
	private String id;

	// 외래 키 매핑시 여러 컬럼을 매핑해야 하므로 JoinColumns을 사용하고
	// 각 외래 키 컬럼은 JoinColumn을 사용
	@ManyToOne
	@JoinColumns({ // referenceColumnName은 name과 같으면 생략 가능
		@JoinColumn(name="PARENT_ID1", referencedColumnName="PARENT_ID1"),
		@JoinColumn(name="PARENT_ID2", referencedColumnName="PARENT_ID2")
	})
	private Parent parent;
}
```

```java
// 저장하기
Parent parent = new Parent();
parent.setId1("pid1");
parent.setId2("pid2");
em.persist(parent);

// 조회하기
ParentId pId = new ParentId();
pId.setId1("pid1");
pId.setId2("pid2");
Parent parent = em.find(Parent.class, pId);
```

- 관계형 데이터베이스에 가까운 방법
- persist()가 호출되어 엔티티가 등록되기 직전에 내부에서 두 개의 id 값을 이용해\
식별자 클래스인 ParendId를 생성하고 영속성 컨텍스트의 키로 사용한다
- 식별자 클래스를 사용해서 엔티티를 조회한다

#### @EmbeddedId 사용하기

```java
// ParentId (식별자 클래스)
@Embeddable // 식별자 클래스
public class ParentId implements Serializable { // Serializable를 구현한 public 클래스
	@Column(name="PARENT_ID1")
	private String id1;

	@Column(name="PARENT_ID2")
	private String id2;

	public ParentId() { } // 기본 생성자

	// equals와 hashCode 메소드 구현	
}
```

```java
// 저장하기
Parent parent = new Parent();
ParentId pId = new ParentId();
pId.setId1("pid1");
pId.setId2("pid2");
parent.setId(pId);
em.persise(parent);

// 조회하기
ParentId pId = new ParentId();
pId.setId1("pid1");
pId.setId2("pid2");
Parent parent = em.find(Parent.class, pId);
```

- 객체지향에 가까운 방법
- 식별자 클래스에 기본 키를 직접 매핑
- JPQL을 작성할 때 @IdClass 사용시 p.id1가\
@EmbeddedId를 사용하면 p.id.id1과 같이 길어질 수 있음

#### equals와 hashCode 메소드가 필요한 이유
- 영속성 컨텍스트는 엔티티의 식별자로 키를 사용해 엔티티를 관리하고\
식별자를 비교할 때 equals와 hashCode를 사용
- 동등성 비교를 위한 메소드가 없으면 예상과 다른 엔티티가 조회되거나 탐색이 불가능
- 따라서 복합 키를 사용하면 이 두 메소드를 필수적으로 구현해야 하고\
이클립스 메뉴에 있는 `Generate hashCode() and equals()...`를 이용하면 편리하다

### 복합키 식별 관계 매핑

#### @IdClass 사용하기

```java
// Parent 클래스
@Entity
public class Parent {
	@Id @Column(name="PARENT_ID")
	private String id;
}

// Child 클래스
@Entity
@IdClass(ChildId.class)
public class Child {
	@Id
	@ManyToOne // 기본 키와 외래 키를 같이 매핑
	@JoinColumn(name="PARENT_ID")
	public Parent parent;

	@Id @Column(name="CHILD_ID")
	private String childId;
}

// ChildId 클래스
public class ChildId implements Serializable {
	private String parent; // Child.parent
	private String childId; // Child.childId

	// equals, hashCode 구현
}

// GrandChild 클래스
@Entity
@IdClass(GrandChildId.class)
public class GrandChild {
	@Id
	@ManyToOne
	@JoinColumns({
		@JoinColumn(name="PARENT_ID"),
		@JoinColumn(name="CHILD_ID")
	})
	private Child child;

	@Id @Column(name="GRANDCHILD_ID")
	private String id;
}

// GrandChildId 클래스
public class GrandChildId implements Serializable {
	private ChildId child; // GrandChild.child
	private String id; // GrandChild.id

	// equals, hashCode 구현
}
```

- 식별 관계는 외래 키, 기본 키를 같이 매핑하기 때문에\
식별자 매핑 `@Id`와 연관관계 매핑 `@ManyToOne`을 같이 사용한다


#### @EmbeddedId 사용하기

```java
// Parent
@Entity
public class Parent {
	@Id @Column(name="PARENT_ID")
	private String id;
}

// Child
@Entity
public class Child {
	@EmbeddedId
	private ChildId id;

	@MapsId("parentId") // ChildId.parentId
	@ManyToOne
	@JoinColumn(name="PARENT_ID")
	public Parent parent;
}

// ChildId
@Embeddable
public class ChildId implements Serializable {
	private String parentId; // @MapsId("parentId")와 매핑

	@Column(name="CHILD_ID")
	private String id;

	// equals, hashCode 구현
}

// GrandChild
@Entity
public class GrandChild {
	@EmbeddedId
	private GrandChildId id;

	@MapsId("childId") // GrandChildId.childId
	@ManyToOne
	@JoinColumns({
		@JoinColumn(name="PARENT_ID"),
		@JoinColumn(name="CHILD_ID")
	})
	private Child child;
}

// GrandChildId
@Embeddable
public class GrandChildId implements Serializable {
	private ChildId childId; // @MapsId("childId")와 매핑

	@Column(name="GRANDCHILD_ID")
	private String id;

	// equals, hashCode 구현
}
```

- @Id 대신에 `@MapsId`를 사용\
외래 키와 매핑한 연관관계를 기본 키에도 매핑할 때 사용됨
- MapsId의 속성 값으로 @EmbeddedId를 사용한 식별자 클래스의 기본 키 필드를 지정

#### 식별, 비식별 관계의 장단점
- 식별 관계
  + 부모 테이블의 기본 키가 자식 테이블로 전파되다가 자식 테이블의 기본 키 컬럼이 점점 증가
  + 식별 관계에서는 2개 이상의 컬럼을 합해서 복합 기본 키를 만들어야 하는 경우가 생김
  + 기본 키로 비즈니스 의미가 있는 자연 키 컬럼을 조합하는 경우가 많음
- 비식별 관계
  + 기본 키로 비즈니스와 관계없는 대리 키를 주로 사용
  + 테이블 구조가 식별 관계보다 유연한 편
  + 복합 키 클래스를 별도로 두지 않아서 편리 (@GeneratedValue 이용)

### 4) 조인 테이블

#### 데이터베이스 테이블 연관관계 설계 방법

- **조인 컬럼 사용** `@JoinColumn`\
테이블간 관계를 조인 컬럼이라 부르는 외래 키 컬럼을 사용하여 관리\
외래 키에 null을 허용하는 선택적 비식별 관계의 경우 외부 조인을 사용해야 함
- **조인 테이블 사용** `@JoinTable`\
조인 테이블이라는 별도의 테이블을 사용해 연관관계를 관리\
테이블을 하나 더 추가해야하는 단점이 있음

### 일대일 조인 테이블

```java
// Parent 클래스 내부
@OneToOne
@JoinTable(name="PARENT_CHILD",
	joinColumns=@JoinColumn(name="PARENT_ID"),
	inverseJoinColumns=@JoinColumn(name="CHILD_ID")
)
private Child child;
```

- 조인 테이블의 외래 키 컬럼 각각에 유니크 제약 조건이 필요
- 부모 엔티티에 `@JoinTable`을 사용
  +  name 속성: 매핑할 조인 테이블 이름
  + joinColumns: 현재 엔티티를 참조하는 외래 키
  + inverseJoinColumns: 반대방향 엔티티를 참조하는 외래 키
- 양방향 매핑시 Child에 `@OneToOne(mappedBy="child")`를 Parent 참조 필드에 지정하면 된다

### 일대다 조인 테이블

```java
// Parent 클래스 내부
@OneToMany
@JoinTable(name="PARENT_CHILD",
	joinColumns=@JoinColumn(name="PARENT_ID"),
	inverseJoinColumns=@JoinColumn(name="CHILD_ID")
)
private List<Child> child = new ArrayList<Child>();
```

- 조인 테이블의 컬럼중 다多와 관련된 CHILD_ID 컬럼에 유니크 제약 조건 부여

### 다대일 조인 테이블

```java
// Parent 클래스 내부
@OneToMany(mappedBy="parent")
private List<Child> child = new ArrayList<Child>();

// Child 클래스 내부
@ManyToOne
@JoinTable(name="PARENT_CHILD",
	joinColumns=@JoinColumn(name="CHILD_ID"),
	inverseJoinColumns=@JoinColumn(name="PARENT_ID")
)
private Parent parent;
```

### 다대다 조인 테이블

```java
// Parent 클래스 내부
@ManyToMany
@JoinTable(name="PARENT_CHILD",
	joinColumns=@JoinColumn(name="PARENT_ID"),
	inverseJoinColumns=@JoinColumn(name="CHILD_ID")
)
private List<Child> child = new ArrayList<child>();
```

- 조인 테이블의 두 컬럼을 합해서\
하나의 복합 유니크 제약 조건을 부여해야 함
- 조인 테이블에 컬럼이 추가되면 @JoinTable을 사용할 수 없게 된다

### 엔티티 하나에 여러 테이블 매핑

```java
@Entity
@Table(name="BOARD")
@SecondaryTable(name="BOARD_DETAIL",
   pkJoinColumns=@PrimaryKeyJoinColumn(name="BOARD_DETAIL_ID"))
public class Board {

	...

	@Column(table="BOARD_DETAIL")
	private String content;
}
```

- `@SecondaryTable`을 사용하면 기존 테이블에 추가적으로 테이블을 매핑할 수 있다\
(@SecondaryTables를 사용하면 더 많은 테이블을 추가)
- name 속성: 매핑할 다른 테이블의 이름\
pkJoinColumns 속성: 매핑할 다른 테이블의 기본 키 컬럼 지정
- 두 테이블 이상을 하나의 엔티티에 매핑하는 방법보다\
테이블 당 엔티티를 각각 만들어 일대일로 매핑하는 것이 권장된다