## HTTP vs HTTPS

### HTTP

HTTP는 HTML과 같은 하이퍼미디어 문서의 전송을 위한 어플리케이션단 프로토콜이다.

웹 서버와 브라우저 간의 통신을 위해 설계되었다.

### HTTPS

`HTTP + Secure`으로, 서버-브라우저로 통하는 정보를 암호화하지 않는 HTML의 보안성 문제를 해결하기 위해 고안되었다.

HTTPS는 **SSL(Secure socket layer) 인증서**를 사용해 브라우저와 서버 사이에 안전한 암호화 연결을 맺어 정보를 안전하게 보호한다.

추가적으로 **TLS(Transport layer security) 프로토콜**을 통해 전송중 데이터의 수정이나 손상을 방지하는 데이터 무결성과 올바른 웹사이트와 통신하고 있음을 증명하는 인증을 제공한다.

[HTTP vs HTTPS: The Difference And Everything You Need To Know](https://seopressor.com/blog/http-vs-https/)

[An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)

## RESTful

REST란 웹에서 컴퓨터 시스템 간에 표준을 제공하기 위한 아키텍처 스타일이다.

하나의 클라이언트와 서버에서 모바일, IoT 등 다양한 시스템으로 분산되기 시작하면서 클라이언트와 서버의 관심 *concern*에 대한 분리의 필요성이 대두되었다.

클라이언트와 서버 구현의 분리는 플랫폼간 인터페이스에 유연성을 제공하고, 간단화된 서버 컴포넌트를 통해 향상된 확장성을 제공한다.

REST 인터페이스를 통해 서로 다른 클라이언트들은 동일한 `Endpoint`에 도달하고, 동일한 `Action`을 수행하며, 동일한 `Response`를 받는다.

> 다음과 같은 기본적인 규칙을 지켜야 한다. 
> 1) URI는 정보의 자원을 표현해야 한다.
> 2) 자원에 대한 행위는 HTTP Method로 표현한다.

### 요청에 포함되는 정보

```html
GET /articles/23
Accept: text/html, application/xhtml
```

- 수행할 작업의 종류를 정의하는 HTTP 동사 *verb*
    - GET(검색), POST(생성), PUT(수정), DELETE(삭제)
- 요청에 대한 정보를 전달할 수 있는 헤더 *header*
    - Accept([콘텐츠 유형](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types))
        - `type/subtype`으로 명시

            예) `text/html`, `image/png`, `application/json`

- 리소스에 대한 경로 *path*
- 데이터를 담을 추가적인 메시지 바디 *body*

### 응답에 포함되는 정보

```html
HTTP/1.1 200 (OK)
Content-Type: text/html
```

- 콘텐츠 유형 헤더 *content-type*
    - 클라이언트가 Accept에서 명시한 옵션 중 하나여야 함
- 응답 코드 *Status Code*
    - `2xx`, `3xx`, `4xx`, `5xx`으로 시작하는 상황에 맞는 상태 코드를 전달

[What is REST?](https://www.codecademy.com/articles/what-is-rest)

## 필수로 알아야 할 HTTP 응답 코드

### 2XX - 성공

- `200 (OK)` : 요청 성공 (GET, PUT)
- `201 (Created)` : 요청이 성공했으며 그로 인한 리소스가 생성됨 (POST)
- `204 (No Content)` : 요청은 성공했으나 응답 바디에 정보가 없음

### 3XX - 리다이렉션

- `301 (Moved Permanetly)` : 요청된 리소스의 URL이 영구적으로 변경되었으며 새 URL이 응답에 제공됨

### 4XX - 클라이언트 오류

- `400 (Bad Request)` : 잘못된 요청 구문 등으로 인해 서버가 요청을 이해할 수 없음
- `401 (UnAuthorized)` : 표준은 '승인되지 않음'이지만 의미상으로는 '인증되지 않음'을 뜻함. 응답을 위해서 사용자의 인증이 필요함을 알림
- `403 (Forbidden)` : 클라이언트가 리소스 접근에 대한 권한이 없음 (401과 달리 서버는 클라이언트의 신분을 알고 있는 상태)
- `404 (Not Found)` : endpoint는 유효하지만 리소스를 찾을 수 없음 (403 대신 리소스의 존재를 숨기기 위해 사용될 수 있음)

### 5XX - 서버 오류

- `500 (Internal Server Error)` : 서버가 처리할 수 없는 상황에 대한 응답

[HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)