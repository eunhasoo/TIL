# Cross Site Scripting (XSS)

### 개요

- `악성 스크립트`가 무해하고 신뢰할 수 있는 사이트에 삽입되는 injection 유형의 공격
- 사용자의 입력을 받아서 유효성 검사나 인코딩없이 그것을 출력하는 애플리케이션에서 공격이 수행됨
- 교차 사이트 스크립팅의 취약성은 공격자가 피해자로 가장하여\
사용자가 할 수 있는 어떠한 행동이든 수행하며, 사용자의 데이터에 접근할 수 있도록 만듬

### 수행 과정

![](https://portswigger.net/web-security/images/cross-site-scripting.svg)

1. 데이터가 신뢰할 수 없는 소스를 통해 웹 애플리케이션에 입력된다.
2. 악성 컨텐츠 유효성 검사를 거치지 않고 응답으로 웹 사용자에게 전송되는 동적 컨텐츠에 포함된다.

웹 브라우저로 전송된 악성 컨텐츠는 대부분 자바스크립트의 조각으로 전달되지만,\
HTML, 플래쉬 혹은 브라우저가 수행할 수 있는 다른 어떤 유형의 코드라도 될 수 있다.\
XSS 기반 공격의 다양성에는 거의 제한이 없지만 보통은 **쿠키나 세션 정보와 같은 프라이빗 데이터를 공격자에게 전달**하는 것,\
**공격자에 의해 통제된 웹 콘텐츠로 피해자를 리다이렉트** 시키는 것,\
**취약한 사이트를 가장하여 유저의 기계에 악의적인 작업을 수행**하는 것 등을 포함한다.

### 공격 유형

#### 1. 저장형 _stored_ XSS 공격

- 애플리케이션이 신뢰할 수 없는 소스로부터 데이터를 얻고나서\
나중에 HTTP 응답 내부에 그 데이터를 포함할 때 발생
- HTTP 요청을 통해, _예) 블로그 포스트의 댓글, 채팅방의 사용자 닉네임, 주문 고객의 연락 정보 등_\
신뢰할 수 없는 소스로부터 _예) 소셜 미디어 포스트를 보여주는 마케팅 애플리케이션, 네트워크 모니터링 애플리케이션 등_\
문제가 되는 데이터가 애플리케이션으로 제출될 수 있음
- 피해자는 **저장된 정보를 요청할 때** 서버로부터 악성 스크립트를 전달받게 됨
- Persistent 혹은 Type-I XSS로도 불림

#### 1-1. 예시

```html
<p>This is My Message.</p>
```

메시지 애플리케이션은 사용자로부터 메시지를 제출받아서 다른 사용자에게 보여준다.\
데이터에 어떠한 가공도 하지 않았기 때문에 공격자는 메시지를 전송해 아래와 같이 쉽게 다른 사용자를 공격할 수 있다.

```html
<p><script>/*doing something bad...*/</script></p>
```


#### 2. 반사형 _reflected_ XSS 공격

- 애플리케이션이 HTTP 요청 안의 데이터를 받아서 즉각적인 응답 안에 해당 데이터를 포함할 때 발생
- 이메일이나 다른 웹 사이트 같이 외부 경로를 통해 피해자에게 전달될 수 있음
- 브라우저는 신뢰할 수 있는 사이트로부터 받은 코드이기 때문에 방어없이 수행
- Non-Persistent 혹은 Type-II XSS라고도 불림

#### 2-1. 예시

```html
<!-- https://website.com/status?message=All+is+well으로 요청 -->

<p>Status: All is well</p>
```

애플리케이션이 데이터에 어떠한 가공도 하지 않았기 때문에 공격자는 쉽게 공격할 수 있다.

```html
<!-- https://website.com/status?message=<script>/*doing something bad...*/</script> 으로 요청 -->

<p>Status: <script>/*doing something bad...*/</script></p>
```

만일 사용자가 브라우저를 통해 공격자에 의해 설계된 URL으로 방문하면\
공격자의 스크립트가 사용자의 애플리케이션 세션 컨텍스트 안에서 수행된다.\
이 때 스크립트는 어떠한 행위든지 수행할 수 있으며, 사용자가 접근할 수 있는 모든 데이터를 검색할 수 있다.

#### 3. DOM 기반 XSS 공격

- 데이터를 DOM에 다시 기록하여 신뢰할 수 없는 소스의 데이터를\
안전하지 않은 방식으로 처리하는 클라이언트 측의 자바스크립트로 인해 발생

#### 3-1. 예시

```javascript
var search = document.getElementById('search').value;
var results = document.getElementById('results');
results.innerHTML = 'You searched for: ' + search;
```

입력 필드값을 읽어 HTML 안에 엘리먼트의 값으로 채우는 애플리케이션이다.\
공격자가 입력 필드의 값을 통제할 수 있게 되면, 그들은 그들의 스크립트를 수행하도록 하는 악성 값을 쉽게 끼워넣을 수 있다.

```javascript
You searched for: <img src=1 onerror='/*doing something bad...*/'>
```

일반적으로, 입력 필드는 URL 쿼리 스트링 파라미터와 같이 HTTP 요청의 일부로 채워지고\
공격자는 반사형 XSS와 같은 방식으로 악성 URL을 사용해 공격할 수 있게 된다.

### 예방 방법

- **도착시 입력 필터링하기**\
사용자 입력이 전달될 때 예상되거나 유효한 입력을 기반으로 최대한 엄격하게 필터링한다
- **출력시 데이터 인코딩하기**\
사용자가 제어 가능한 데이터가 HTTP 응답으로 출력되는 경우 인코딩을 통해 코드로 해석되지 않도록 한다
- **적절한 응답 헤더 사용하기**\
`Content-Type`, `X-Content-Type-Options` 헤더를 사용해 의도대로 브라우저가 응답을 해석하도록 확실히 한다
- **콘텐츠 보안 정책** _Content Security Policy_ \
CSP를 사용해 여전히 발생하고 있는 어떠한 XSS 취약성이든 발생 가능성을 줄인다
- **라이브러리 사용**\
[오픈 소스 인코딩 라이브러리](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html#inconsistencies-of-encoding-libraries)를 사용한다

#### 참고
- [Cross-site scripting](https://portswigger.net/web-security/cross-site-scripting)
- [Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [DOM Based XSS](https://owasp.org/www-community/attacks/DOM_Based_XSS)
- [XSS (Cross Site Scripting) Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)