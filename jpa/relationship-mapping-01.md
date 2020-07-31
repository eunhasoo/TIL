# 연관관계 매핑 기초

### 1) 단방향 연관관계

#### 다대일 Many-to-One

- 회원과 팀이 존재할 때\
`회원`은 `하나의 팀`에만 속함
- 이때 `회원`과 `팀`은 `다대일` 관계

#### 객체 연관관계 vs 테이블 연관관계

- 객체 `참조`를 통한 연관관계는 언제나 **단방향**이지만\
참조 필드를 다른 쪽에 추가함으로써 양방향같은 단방향 관계 2개를 만들 수 있음
- 테이블은 `외래 키` 하나로 **양방향**에서 `조인`할 수 있음

#### 객체 관계 매핑

```java
// Member 클래스
@ManyToOne
@JoinColumn(name="TEAM_ID")
private Team team;

// Team 클래스
@Id
@Column(name="TEAM_ID")
private String id;

// Service
Team team = new Team("1", "soccer");
Member member = new Member("1", "kim");
member.setTeam(team); // 팀 참조
em.persist(member); // 엔티티 저장
```

- `@ManyToOne`: 다대일 관계라는 매핑 정보. (필수)
- `@JoinColumn`: name 속성으로 매핑할 외래 키 이름을 지정. (생략 가능)

#### 객체 관계 조회 방법 2가지

1. 객체 그래프 탐색 (객체 연관관계 사용)
2. 객체지향 쿼리 사용 _JPQL_

#### JPQL을 사용한 조회

```java
String jpql = "SELECT m FROM Member m JOIN m.team t where " + "t.name=:teamName";

List<Member> members = em.createQuery(jpql, Member.class)
    			.setParameter("teamName", "soccer")
    			.getResultList();
for (Member member : members) {
	System.out.println("username: " + member.getUsername());
}
```

- SQL으로 연관 테이블을 조인해 조회하듯이 JPQL이 지원하는 문법을 사용한다
- `:teamName`은 파라미터로 전달된 값을 바인딩한다
- SQL과 달리 객체(엔티티)를 대상으로 하며 더 간결하다

#### 연관관계 수정

```java
Team team2 = new Team("2", "baseball");

Member member = em.find(Member.class, "1");
System.out.println(member.getTeam().getName()); // soccer

member.setTeam(team2);
System.out.println(member.getTeam().getName()); // baseball
```

- 단순히 불러온 엔티티의 값만 변경하면\
트랜잭션이 커밋될 때 플러시가 일어나면서 자동으로 값을 변경하고 데이터베이스에 반영되듯이\
연관관계에서도 참조대상만 변경하면 마찬가지로 JPA가 알아서 자동으로 처리한다

#### 연관관계 제거

```java
Team team = em.find(Team.class, "1");
em.remove(team); // error. 연관관계를 먼저 끊어야 한다

Member member1 = em.find(Member.class, "1");
em.setTeam(null); // 연관관계 제거
em.remove(team); // 정상적으로 수행된다
```

- 연관관계로 매핑된 필드를 null으로 지정해주면 연관관계가 제거된다
- 연관된 엔티티를 삭제하기 위해서는 **외래 키 제약조건**으로 인해\
기존에 있던 연관관계를 모두 제거하고 삭제해야 한다

### 2) 양방향 연관관계

#### 일대다 One-to-Many

- 회원과 팀이 `다대일` 관계일 때\
반대로 팀에서 회원은 `일대다` 관계
- 회원 → 팀: `member.team`\
팀 → 회원: `team.members`
- 일대다 관계는 Collection 객체를 이용해야한다

#### 객체 관계 매핑

```java
// Member 클래스
@ManyToOne
@JoinColumn(name="TEAM_ID")
private Team team;

// Team 클래스
@OneToMany(mappedBy="team") // team은 연관관계의 주인인 Member의 team 필드를 의미
private List<Member> members = new ArrayList<Member>();
```

- `@OneToMany`: 일대다 매핑 정보
- `mappedBy`: 반대쪽 매핑의 필드 이름을 적어주면 된다

#### 연관관계의 주인

- 양방향 매핑에서는 두 연관관계 중 하나를 연관관계의 주인 _Owner_ 으로 정해야 함
- 연관관계 주인은 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있고\
주인이 아니면 읽기만 할 수 있다
- 주인을 정한다는 것은 **외래 키 관리자를 선택하는 것**\
즉, 외래 키가 있는 곳 _테이블_ (**다**대일/일대**다**의 다 쪽)으로 정해야 한다
- 주인이 아닌 곳에는 `mappedBy` 속성의 값으로 연관관계의 주인을 지정

#### 순수한 객체까지 고려한 양방향 연관관계

```java
Team team = new Team("1", "soccer");
em.persist(team);

Member member = new Member("1", "kim");

// 주인이 아닌 쪽까지 양방향으로 값을 입력해주는 것이 가장 안전하다
member.setTeam(team); // 연관관계 설정(주인)
team.getMembers().add(member); // 연관관계 설정

em.persist(member);
```

#### 문제점

```java
// team 설정
member.setTeam(team1);
team1.getMembers().add(member);

// team 변경
member.setTeam(team2);
team2.getMembers().add(member);
Member findMember = team1.getMembers().get(0); // team1 내에 member가 여전히 존재
```

- team1에서 team2로 변경했지만 연관관계는 제거되지 않았음
- 새로운 영속성 컨텍스트에서는 변경이 반영되지만\
기존 영속성 컨텍스트가 살아있는 상태에서는 member가 그대로 반환됨
- 기존 team과 member의 연관관계를 제거하는 코드를 추가해줘야 함

```java
// Member 클래스
public void setTeam(Team team) {
	if (this.team != null) // 기존 관계 제거
		this.team.getMembers().remove(this);
	this.team = team;
	team.getMembers().add(this); // 한번에 양방향 설정
}
```
