# 값 타입

### 1) 엔티티 타입

- 식별자 `@Id` 가 있어서 구별될 수 있다
- 생성, 영속화, 소멸의 생명 주기가 있다
- 공유할 수 있다 (공유 참조)

### 2) 기본 값 타입

- 식별자 값이 없고\
회원 엔티티에 생명주기를 의존하고 있음
- 엔티티 인스턴스를 제거하면 값도 제거됨
- 공유하지 않고 불변 객체로 만드는 것이 안전함

### 3) 임베디드 (복합 값) 타입

```java
// Member 클래스 내부
@Embedded
Period workPeriod; // 근무 기간. 근무 시작일과 종료일을 가짐.
@Embedded
Address homeAddress; // 집 주소. 우편번호, 도시명, 번지수 등을 가짐

// Period 클래스
@Embeddable
public class Period {
	@Temporal(TemporalType.DATE) Date startDate;
	@Temporal(TemporalType.DATE) Date endDate;
	...
}

// Address 클래스
@Embeddable
public class Address {
	private String city;
	private String street;
	private String zipcode;
	...
}
```

- 원래는 값 타입으로 나열되어있던 정보들을\
새로운 타입으로 정의하여 하나로 묶음으로써 명확한 코드가 됨
- 더욱 의미있고 응집력 있게 타입이 정의될 수 있음
- `기본 생성자`를 필수로 가져야 하고,\
값 타입이 정의되는 곳에 `@Embeddable`를 붙이고\
값 타입이 사용되는 곳에 `@Embedded`를 붙여야 함
- 임베디드 타입이 null으로 설정되면 매핑된 컬럼값은 모두 null이 된다\
예) member.setAddress(null) 이면 city, street, zipcode 값은 모두 null

#### 속성 재정의

```java
// 기존 집 주소
@Embedded
Address homeAddress;

// 회사 주소 추가
@Embedded
@AttributeOverrides({
	@AttributeOverride(name="city", column=@Column(name="COMPANY_CITY")),
	@AttributeOverride(name="street", column=@Column(name="COMPANY_STREET")),
	@AttributeOverride(name="zipcode", column=@Column(name="COMPANY_ZIPCODE"))
})
Address companyAddress;
```

- 같은 구조의 컬럼이 하나 더 필요할 때\
`@AttributeOverride`를 통해서 매핑 정보를 재정의
- 임베디드 타입 내부에 임베디드 타입을 갖고 있더라도\
이 어노테이션은 엔티티에 설정해야 한다

### 4) 값 타입의 공유 참조

```java
member1.setHomeAddress(oldcity);
Address address = member1.getHomeAddress(); // 회원 1에 oldCity 설정

address.setCity(newCity);
member2.setHomeAddress(address); // 회원 2를 설정하다가 회원 1도 newCity로 변경되어버렸다
```

- 값 타입을 여러 엔티티에서 공유 참조하는 것은 위험하다
- 값을 (새로운 인스턴스로) 복사해서 사용하는 것이 안전하다
- 기본 타입은 값이 복사되지만 객체 타입은 참조 값을 전달하기 때문에\
공유 참조로 인한 부작용을 피할 수 없다

#### 불변 객체

- 값을 조회할 수는 있지만 수정은 할 수 없는 객체
- 불변 객체로 만드는 법은 간단하게 생성자로만 값을 설정하도록 하고\
수정자(setter)를 만들지 않으면 된다
- 값을 수정해야 할 때에는 새로운 객체를 생성해서 사용해야 하지만\
공유 참조라는 부작용을 막을 수 있는 좋은 방안이다

#### 동등성 비교

```java
Address address1 = new Address("서울시", "종로구", "1번지");
Address address2 = new Address("서울시", "종로구", "1번지");
address1.equals(address2); // true
```

- 동등성 _equivalence_ 비교를 할 때 두 객체의 값은 같아야 한다
- 보통 모든 필드 값을 비교해서 동등성을 비교하도록 equals 메소드를 재정의한다
- hashCode 메소드도 같이 재정의 하는 것이 안전하다

### 5) 값 타입 컬렉션

```java
@Entity
public class Member {
	@Id @GeneratedValue
	private Long id;

	@Embedded 
	private Address address;

	@ElementCollection
	@CollectionTable(name="FAVORITE_FOODS",
		joinColumns=@JoinColumn(name="MEMBER_ID"))
	@Column(name="FOOD_NAME")
	private Set<String> favoriteFoods = new HashSet<String>();

	@ElementCollection
	@CollectionTable(name="ADDRESS", joinColumns=@JoinColumn(name="MEMBER_ID"))
	private List<Address> addressHistory = new ArrayList<Address>();
	...
}
```

- 값 타입을 하나 이상 저장하려면 컬렉션 객체를 사용해서\
`@ElementCollection`, `@CollectionTable`을 붙이면 된다
- 엔티티를 저장할 때는 member 엔티티 생성후 값 타입을 모두 설정set하고\
member 엔티티만 persist() 호출을 통해 영속화하면 된다
- 컬렉션에 저장된 값의 개수만큼 INSERT SQL이 실행된다

#### 제약사항

- 특정 엔티티에 속한 값 타입은 변경되더라도\
소속된 엔티티를 DB에서 찾아 값을 변경하면 된다
- 값 타입 컬렉션에 보관된 값 타입들은 별도의 테이블에 보관되기 때문에\
**값이 변경되면 DB에 있는 원본 데이터를 찾기 어려운 문제**가 발생한다
- JPA 구현체는 대부분 값 타입 컬렉션에 변경이 발생하면\
값 타입 컬렉션이 매핑된 테이블의 연관된 모든 데이터를 삭제하고\
현재 객체에 저장된 값들을 모두 다시 저장한다
- 실무에서 값 타입 컬렉션에 매핑된 테이블에 데이터가 많다면\
값 타입 컬렉션 대신 `일대다` 관계 사용을 고려하는게 좋다
- 일대다 관계에 추가적으로 영속성 전이 + 고아 객체 제거 기능을 적용하면\
값 타입 컬렉션처럼 사용이 가능하다

