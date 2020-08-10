# 의존성 주입 패턴

### 의존성 주입

- `@Repository`, `@Service` 등 Component Scan의 대상이 되는\
어노테이션이 붙은 클래스는 스프링이 알아서 빈으로 등록
- 등록된 빈들은 `@Autowired` 등의 어노테이션을 통해 의존성을 주입받을 수 있음
- <참고>
  + [Spring DI Patterns: The Good, The Bad, and The Ugly](https://dzone.com/articles/spring-di-patterns-the-good-the-bad-and-the-ugly)
  + [스프링 JPA 인프런 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1)

### 1) 필드 주입

```java
@Service
public class XxxxService {

	@Autowired
	private XxxxRepository xxxxRepository;

	// Business logic..
}
```

- 가장 간편하게 구현할 수 있지만 비권장되는 방법
- 테스트를 할 때 접근/주입하기 난해함
- final으로 지정할 수 없음
- 순환 종속성 문제 발생 가능

### 2) 수정자 주입

```java
@Service
public class XxxxService {

	private XxxxRepository xxxxRepository;
	
	@Autowired
	public void setXxxxRepository(final XxxxRepository xxxxRepository) {
		this.xxxxRepository = xxxxRepository;
	}

	// Business logic..
}
```

- 순환 종속성 문제에 영향을 받지 않음
- 결합된 클래스 식별이 명확함
- 런타임 시점에 누군가로부터 변경될 가능성이 있는데\
대부분 초기 세팅 이후 변경될 필요가 없는 경우가 많음


### 3) 생성자 주입

```java
@Service
public class XxxxService {

	private final XxxxRepository xxxxRepository;
	
	// @Autowired 생략 가능
	public XxxxService(final XxxxRepository xxxxRepository) {
		this.xxxxRepository = xxxxRepository;
	}

	// Business logic..
}
```

- 앞선 방법들중 가장 이상적인 방법
- 생성과 동시에 불변성을 가짐
- 테스트 작성할 때 편리하고, 실수로 인한 런타임 오류 발생을 쉽게 알아챌 수 있음
- 생성자가 하나면 `@Autowired` 생략이 가능하다

#### lombok을 적용해서 편리하게 구현

```java
// @AllArgsConstructor 
@RequiredArgsConstructor
@Service
public class XxxxService {

	private final XxxxRepository xxxxRepository;

	// Business logic..
}
```

- `@AllArgsConstructor` 사용도 가능하지만\
`@RequiredArgsConstructor`가 더 나은 방법
- <참고> [lombok constructor 어노테이션](https://projectlombok.org/features/constructor)