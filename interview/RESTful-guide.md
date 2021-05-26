# RESTful API 설계 가이드

## Richardson Maturity Model

- REST 서비스의 성숙도를 식별하기 위한 모델
- REST를 잘 따르고 있는가를 기준으로 Leonard Richardson이 네 가지 카테고리로 분류
	- 결정을 위한 세 가지 요소로 `URI`, `HTTP 메소드`, `HATEOAS`를 사용

![](https://i1.wp.com/restfulapi.net/wp-content/uploads/Richardson-Maturity-Model.jpg?w=600&ssl=1)

### Level 0
- 하나의 URI와 HTTP 메소드만을 가짐 (거의 POST)
- 대부분의 웹 서비스가 속하는 레벨

### Level 1
- 적절한 URI를 통해 리소스를 제공
- 적절한 HTTP 메소드와 HATEOAS는 사용하지 않음
	- 일반적으로 POST 메소드 이용

### Level 2
- Level 1 + 적절한 HTTP 메소드를 사용
	- GET, POST, DELETE, PUT

### Level 3
- Level 2 + HATEOAS 까지 사용
- 가장 성숙한 레벨으로, 쉽게 검색할 수 있도록 함
- HATEOAS로 인해 스스로 설명될 수 있는 응답을 생성
	- 이어서 할 수 있는 작업의 action, URI을 제공
- 최소한의 endpoint로 클라이언트에서 다음 리소스의 경로를 확인가능
 
## Best practice
- Consumer를 우선하기
- HTTP의 장점을 최대화하기
	- 헤더값, 응답 타입, HTTP 메소드 등
- 적절한 HTTP Method 제공하기
	- GET, POST, PUT, DELETE
- 적절한 Response Status 반환하기
	- 200, 404, 400, 201 등...
- URI에 기밀 정보를 노출하지 않기
- 복수형 사용하기
	- `/user` 대신 `/users`
	- `/user/1` 대신 `/users/1`
- 동사형 대신 명사형으로 표현하기
	- 간단하고 명료하게 명시 가능
- 일관된 endpoint 사용하기
	- `PUT` /gists/{id}/star
	- `DELETE` /gists/{id}/star
		- 하나의 endpoint를 사용해서 클라이언트가 실행하려는 작업은 HTTP Method로 선택

### 참고
- [Richardson Maturity Model](https://restfulapi.net/richardson-maturity-model/)
- [Spring Boot를 이용한 RESTful Web Services 개발](https://www.inflearn.com/course/spring-boot-restful-web-services/dashboard)