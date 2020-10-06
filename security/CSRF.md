
# CSRF

### 1. CSRF란?

교차 사이트 요청 위조 _Cross Site Request Forgery_ 를 의미한다.\
현재 `A은행`에 로그인한 사용자가 `B은행` 계좌로 이체하는 상황을 생각해보자.

```html
<form method="post" action="/transfer">
  <input type="text" name="amount"/>
  <input type="text" name="routingNumber"/>
  <input type="text" name="account"/>
  <input type="submit" value="Transfer"/>
</form>
```

위와 같은 양식을 통해 폼을 전송한다고 할 때 전달되는 HTTP 요청은 아래와 같다. 

```
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100000&routingNumber=1234&account=9876
```

그렇다면 이제 로그아웃하지 않은 채(인증된 상태)로 악성 웹사이트에 접속했다고 가정하자.\
악성 웹사이트는 다음과 같은 HTML 양식을 가지고 있다.

```html
<form method="post" action="https://bank.example.com/transfer">
  <input type="hidden" name="amount" value="100000"/>
  <input type="hidden" name="routingNumber" value="evilsRoutingNumber"/>
  <input type="hidden" name="account" value="evilsAccountNumber"/>
  <input type="submit" value="돈을 받아가세요!"/>
</form>
```

만일 내가 돈을 받고싶어서 버튼을 누르면, 내 의도와 무관하게 10만원을 공격자에게 보내게 된다.\
공격자는 내 쿠키를 보지 못하지만, 요청과 함께 여전히 `은행과 관련된 쿠키`가 전송되기 때문에 생기는 일이다.\
내가 버튼을 클릭하지 않더라도 자바스크립트를 사용하면 이런 작업이 자동적으로 일어날 수 있고,\
심지어 신뢰할 수 있는 사이트를 방문해도 쉽게 [XSS 공격](https://owasp.org/www-community/attacks/xss/)의 피해자가 될 수 있다.

### 2. CSRF 공격에 방어하기

CSRF 공격이 가능한 이유는 `피해자`의 웹사이트로부터의 요청과 `공격자`의 웹사이트로부터의 요청이 같기 때문이다.\
즉 은행 사이트로부터 오는 요청은 허락하면서도 악성 웹사이트로부터 오는 요청은 거부할 수 있는 방법이 없다는 의미다.\
CSRF 공격에 방어하기 위해서는, 악성 웹사이트의 요청에서는 _제공할 수 없는_ 무엇인가를 통해 우리가 두 요청을 구별지을 수 있어야 한다.

#### 주요 두 가지 방어 메카니즘

1. Synchronizer Token Pattern
2. 세션 쿠키에 SameSite 속성 지정하기

#### 1) Synchronizer Token Pattern

- 세션 쿠키 외에도 `CSRF 토큰`이라고 불리는 랜덤으로 생성된 보안값이 요청 안에 반드시 존재하도록 하고\
이를 매 요청시마다 확인하는, 가장 보편적이고 포괄적인 방법
- 요청을 받으면 서버는 예상되는 CSRF 토큰인지 확인하고, 실제 CSRF 토큰과 비교해서 일치하지 않는다면 요청을 거부해야 함
- 키포인트는 _브라우저로부터 자동적으로 포함되지 않는_ 요청의 일부에 실제 CSRF 토큰이 포함되어야 한다는 것\
(예) 쿠키가 아닌 HTTP 파라미터나 헤더에서 CSRF 토큰을 요구하기
- 어플리케이션의 상태를 업데이트하는 요청에만 CSRF 토큰을 요구하면 보안을 지키면서도 편의성을 향상시킬 수 있음

```html
<form method="post" action="/transfer">
  <input type="hidden" name="_csrf" value="4bfd1575-3ad1-4d21-96c7-4ef2d9f86721"/>
  <input type="text" name="amount"/>
  <input type="text" name="routingNumber"/>
  <input type="hidden" name="account"/>
  <input type="submit" value="Transfer"/>
</form>
```

요청을 위한 폼 양식에 CSRF 토큰이 추가됐다.\
[동일 출처 정책](https://en.wikipedia.org/wiki/Same-origin_policy5) _Same-origin policy_ 으로 인해 외부 사이트에서 CSRF 토큰을 읽어들이지 못한다.


```
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876&_csrf=4bfd1575-3ad1-4d21-96c7-4ef2d9f86721
```

요청 내부에 `_csrf` 파라미터가 추가되었다.\
악성 사이트에서는 이 파라미터에 올바른 값을 제공할 수 없고,\
예상된 CSRF 토큰과 실제 CSRF 토큰의 비교에 실패하면 요청이 수행되지 않는다.

#### 2) SameSite Attribute

- [RFC 6265](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)에서 정의된 속성
- 쿠키를 동일 사이트나 자사 컨텍스트에 한해 제한할 것인지를 명시

```
Set-Cookie: JSESSIONID=randomid; Domain=bank.example.com; Secure; HttpOnly; SameSite=Lax
```

응답 헤더에 `SameSite` 속성이 있음을 확인할 수 있다.\
이 속성에 유효한 값은 `Strict`, `Lax`, `None` 이다.\
`Strict`는 [동일한 사이트](https://tools.ietf.org/html/draft-west-first-party-cookies-07#section-2.1) (유저 입장에서 현재 브라우저의 URL과 매치하는 사이트)로부터 오는 요청에만 쿠키를 포함한다.\
`Lax`는 동일 사이트 혹은 최상위 수준 탐색 _top-level navigations_ 으로부터 오는 요청이거나, * 멱등한 메소드인 경우 쿠키가 포함된다.\
`None`으로 지정하는 경우에는 의도적으로 제 3자 컨텍스트에 쿠키를 전송하기를 원한다고 명시하는 것이다.

> 멱등 Idempotent 하다는 것은 HTTP 메소드가 GET, HEAD, OPTIONS, TRACE인 경우 어플리케이션의 상태를 바꾸어서는 안됨을 의미한다.

![](https://webdev.imgix.net/samesite-cookies-explained/samesite-none-lax-strict.png)

**SamsSite 속성을 사용하는 경우 고려사항**
1. `Strict`으로 지정하는 경우, 강력한 방어가 제공되지만 사용자에게는 혼란을 야기할 수 있다.\
예를 들면 페이스북에 로그인이 유지된 사용자가 구글에서 페이스북 링크가 포함된 이메일을 받아 해당 링크를 클릭하면,\
사용자는 페이스북에 인증되어 있음을 기대하지만 쿠키가 전송되지 않아 사용자가 인증되지 않는다.
2. 브라우저가 반드시 SameSite 속성을 지원해야만 한다.\
최신 브라우저들은 대개 이를 지원하지만, 이전의 브라우저들은 여전히 그렇지 않다.
3. SameSite 속성을 생략하면 `SameSite=Lax`로써 처리된다.\
브라우저에 의존하는 것보다는 명시적으로 지정해주는 것이 권장된다.
4. `SameSite=None`으로 지정하는 경우 `Secure` 속성을 함께 명시해야 한다.\
둘은 한 쌍처럼 명시되어야 한다. Secure 속성을 빼먹을 경우 쿠키가 거부될 수 있다.
5. SameSite 속성은 CSRF 공격을 대비한 단일 방어보다는 심층 방어로써 이용하는 것이 권장된다.

### 3. CSRF 방어가 필요한 경우

일반 사용자들이 브라우저에서 처리한 모든 요청에 대해 CSRF 공격을 방어하는 것이 권장된다.\
_브라우저가 아닌_ 클라이언트에 제공하는 서비스를 만드는 경우에만 비활성화하는 것이 적당하다.

### 4. 구현시 고려사항

#### 로그인

- 로그인 위조에 방어하기 위해 로그인 요청은 CSRF 공격으로부터 보호되어야 함은 물론이고,\
악성 유저가 피해자의 민감한 정보를 읽어들일 수 없도록 위조 로그인 요청에 대한 방어가 필요하다
- 다음과 같이 공격이 실행된다
  - 악성 유저는 악성 유저의 자격증명을 사용해 CSRF 로그인을 수행한다. 피해자는 악성 유저로서 인증된다.
  - 악성 유저는 피해자를 속여 잘못된 웹사이트를 방문하도록 한뒤 민감한 정보를 입력하게 유도한다.
  - 정보가 악성 유저의 계정에 연결되어 있으므로 악성 유저가 그들 자신의 자격증명으로 로그인하여\
피해자의 민감한 정보를 볼 수 있다.

#### 로그아웃

- 위조 로그아웃을 방어하기 위해 로그아웃 요청은 CSRF 공격으로부터 보호되어야 한다

#### 세션 타임아웃

- 예상되는 CSRF 토큰은 세션에 저장되곤 하는데,\
이 방법은 세션이 만료되는 순간 서버가 CSRF 토큰을 찾을 수 없고 요청이 거절될 수 있다
- 이러한 시간제한 문제를 해결하기 위한 옵션은 여러가지가 있다
  - 가장 좋은 방법은 자바스크립트를 사용해 CSRF 토큰을 폼 제출시 함께 요청에 담는 것이다.
  - 다음 옵션은 세션 만료를 사용자에게 알리는 자바스크립트를 작성하는 것이다. \
사용자는 작업을 계속하기 위해 버튼을 클릭해서 세션을 새로고침할 수 있다.
  - 마지막으로는 예상된 CSRF 토큰이 쿠키에 저장될 수 있게 하는 것이다. 이는 예상 CSRF 토큰이 세션보다 오래 지속되도록 한다.\
토큰이 쿠키에 저장되는 것이 디폴트가 아닌 이유는 다른 도메인에서 헤더(쿠키)를 지정해 악용할 수 있고,\
시간 초과와 같이 상태를 잃게 되면 토큰이 손상된 경우 연결을 끝내도록 강제할 수 있는 능력을 잃게 되기 때문이다.

#### Multipart (파일 업로드)

- CSRF 공격으로부터 파일 업로드 요청을 방어하는 것은 닭과 계란 문제가 발생한다.\
CSRF 공격을 방지하기 위해서는 실제 CSRF 토큰을 얻기 위해 요청의 본문 _body_ 를 읽어야 하는데,\
본문을 읽는다는 것은 파일이 업로드된다는 것이고 이는 외부 사이트에서 파일을 업로드 할 수 있다는 것이 된다.
- CSRF 토큰의 위치를 어디에 두냐에 따라 해결 방법이 나뉠 수 있다.
  - **Body**에 토큰을 두는 방법은
    - 인증이 수행되기 전에 본문을 읽어들이므로 누구든 내 서버에 임시 파일을 저장할 수 있음
    - 어플리케이션에서 처리되는 파일은 오직 인증된 사용자만이 제출할 수 있음\
즉 반대로 얘기하면 오직 인증된 사용자가 제출한 파일만이 어플리케이션에서 처리됨
    - 이런 임시 파일 업로드는 대부분 서버에 거의 영향을 미치지 않으므로 권장되는 접근 방식이 됨
  - **URL**에 토큰을 두는 방법은
    - 비인증된 사용자가 임시 파일을 업로드 하는 것을 막고자 하는 경우 대안적으로 사용
    - 예상되는 CSRF 토큰을 폼 속성의 액션 URL의 쿼리 파라미터로 지정한다
    - 이 접근 방식의 단점은 쿼리 파라미터는 유출될 수 있다는 점인데,\
민감한 데이터는 요청의 본문이나 헤더에 둠으로써 유출되지 않도록 하는 것이 모범적인 관례로 여겨진다. [(참고)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec15.html#sec15.1.3)\
따라서 이 방법은 Body에 담는 방식보다 비권장된다

### 참고 및 추가 자료
- [Spring Security Reference](https://docs.spring.io/spring-security/site/docs/5.3.4.RELEASE/reference/html5/#csrf)
- [OWASP CSRFGuard](https://owasp.org/www-project-csrfguard/)
- [SameSite cookies explained](https://web.dev/samesite-cookies-explained/)
- [Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)