# OAuth 2

### 간단한 정의

페이스북, 깃허브와 같은 HTTP 서비스의 사용자 계정에 제한적으로 접근할 수 있도록 하는 인증 _authorization_ 프레임워크이다.\
사용자 계정을 호스팅하는 서비스에 사용자 인증을 위임하고, 제3자 애플리케이션이 사용자 계정에 접근할 수 있도록 권한을 부여한다

### 역할

#### 1. Resource Owner: 사용자

자원 소유자 _Resource Owner_ 는 자신의 계정에 접근하고자 애플리케이션에 인증하는 사용자이다.\
사용자 계정에 대한 애플리케이션의 접근은 부여된 권한 범위에 국한된다. (ex. 읽기/쓰기 권한)

#### 2. Resource / Authorization Server: API

자원 서버 _Resource Server_ 는 사용자 계정을 호스팅한다.\
인증 서버 _Authorization Server_ 는 사용자를 식별하고 애플리케이션에 액세스 토큰 _access token_ 을 발행한다.\
애플리케이션 개발자의 관점에서 서비스의 **API**는 자원, 인증 서버 두 역할을 모두 수행한다.
> 두 역할을 결합해 서비스 혹은 API 역할이라고도 한다.

#### 3. Client: 애플리케이션

클라이언트 _Client_ 는 사용자 계정에 접근을 원하는 애플리케이션이다.\
계정에 접근하기 이전에 사용자에 의해 승인되어야 하며, API를 통해 인증이 검증되어야 한다.

### 추상적인 프로토콜 흐름

![](https://assets.digitalocean.com/articles/oauth/abstract_flow.png)

1. _애플리케이션_ 은 _사용자_ 에게 서비스 자원에 접근할 수 있는 권한을 요청한다.
2. _사용자_ 가 승인하면, _애플리케이션_ 은 권한 부여 Authorization Grant를 얻는다.
3. _애플리케이션_ 은 _인증 서버_ (API)에 액세스 토큰을 요청한다. (자체 ID 인증 및 권한 부여를 제공하면서)
4. 분명한 애플리케이션 ID이고 Authorization Grant가 유효한 경우, _인증 서버_ 는 애플리케이션에 액세스 토큰을 발행한다. 인가가 완료된다.
5. _애플리케이션_ 은 _자원 서버_ (API)에 인증을 위한 액세스 토큰을 제공과 함께 자원을 요청한다.
6. 액세스 토큰이 유효하면 _자원 서버_ 는 _애플리케이션_ 에 자원을 제공한다.

> 이 과정에서 실제 흐름은 사용되는 권한 부여 타입에 따라 다를 수 있다.

### 애플리케이션 등록

OAuth를 사용하기 전에 서비스에 내 애플리케이션을 등록하는 절차가 필요한데,\
서비스 웹사이트 내 개발자 혹은 API 관련된 부분에서 이를 수행할 수 있다. 아래와 같은 정보들이 필요할 수 있다.
- 애플리케이션 이름
- 애플리케이션 웹사이트
- Redirect URI/Callback URL
  - Redirect URI는 인증을 수행한 (혹은 거부당한) 사용자를 리다이렉트해줄 위치

#### Client Identifier, Client Secret

애플리케이션이 등록되고 나면, 서비스는 Client Identifier와 Client Secret의 형태로 Client Credentials을 발행해준다.\
Client Identifier는 서비스 API에서 애플리케이션을 식별하기 위해 공개적으로 노출되는 문자열이며, 사용자에게 제공되는 인증 URL을 빌드하는 데에도 사용된다.\
Client Secret은 애플리케이션이 사용자의 계정에 대한 접근을 서비스 API에게 요청했을 때 애플리케이션의 식별을 증명하는 데에 사용되고, API와 애플리케이션 사이에서 프라이빗하게 유지되어야 한다.

### Grant

인증 서버로부터 액세스 토큰을 얻는 방법을 Grant라고 한다.\
4개의 부여 유형이 존재하고, 각 유형별로 유용한 케이스가 다르다.

- `Authorization Code`: 서버측 애플리케이션에서 사용
- `Implicit`: 모바일 앱 또는 (사용자의 기기에서 실행되는) 웹 애플리케이션에서 사용
- `Resource Owner Password Credentials`: 서비스 자체 소유이거나 신뢰할 수 있는 애플리케이션에서 사용
- `Client Credentials`: 애플리케이션 API 액세스에 사용

### Grant 유형 1. Authorization Code

**가장 일반적으로 사용되는** 유형이다. 소스 코드가 공개적으로 노출되지 않고, Client Secret의 기밀이 유지될 수 있는 서버측 애플리케이션에 최적이기 때문이다.\
이는 **리다이렉션 기반** 플로우이다. 즉, 애플리케이션이 웹 브라우저같은 사용자 에이전트와 상호작용하거나 사용자 에이전트를 통해 라우팅되는 API Authorization Code를 수신할 수 있어야 함을 의미한다.

#### Authorization Code 흐름

![](https://assets.digitalocean.com/articles/oauth/auth_code_flow.png)

#### 1단계: 링크 제공

```url
https://abc.com/v1/oauth/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
```

- 링크 컴포넌트 설명
  - `https://abc.com/oauth/authorize`: API 인증 endpoint
  - `client_id`: API가 애플리케이션을 식별하기 위한 애플리케이션의 클라이언트 ID
  - `response_type`: 애플리케이션이 사용하고자 하는 Grant 유형 (Authorization Code Grant, Implicit Grant ONLY)
  - `scope`: 애플리케이션이 요청할 접근의 수준을 명시

#### 2단계: 사용자는 애플리케이션에 권한을 승인한다

사용자는 링크에 들어가서 ID 인증을 위해 서비스에 로그인을 해야 한다.\
로그인후 서비스는 사용자에게 그들의 계정에 대한 애플리케이션의 접근에 대한 의사를 묻는 메시지를 표시한다.

#### 3단계: 애플리케이션이 Authorization Code를 받는다

사용자가 애플리케이션에 권한을 승인하면, 서비스는 Authorization Code가 달린 애플리케이션 리다이렉트 URI으로 사용자 에이전트를 이동시킨다.\
(이 리다이렉트 주소는 클라이언트 등록 절차에서 명시한 URI이다)

```
https://dropletbook.com/callback?code=AUTHORIZATION_CODE
```

#### 4단계: 애플리케이션이 액세스 토큰을 요청한다

애플리케이션은 client secret을 포함한 인증 세부 정보와 Authorization Code를 넘기면서 API에 액세스 토큰을 요청한다.

```
https://abc.com/v1/oauth/token?client_id=CLIENT_ID&client_secret=CLIENT_SECRET&grant_type=authorization_code&code=AUTHORIZATION_CODE&redirect_uri=CALLBACK_URL
```

#### 5단계: 애플리케이션이 액세스 토큰을 받는다

인증이 유효하다면 API는 아래와 같은 액세스 토큰을 포함한 응답을 애플리케이션에 보낼 것이다.

```json
{
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{
	  "name":"Mark E. Mark",
	  "email":"mark@thefunkybunch.com"
  }
}
```

인증 절차가 완료되었다.\
액세스 토큰은 만료/취소되지 않는 한, 서비스 API를 통해 접근 수준 안에서 사용자 계정에 접근하기 위해 사용되어질 것이다.

### Grant 유형 2. Implicit

**client secret 기밀성이 보장되지 않는** 모바일 앱, 웹 애플리케이션(ex. 웹 브라우저에서 실행되는 애플리케이션)에서 사용되는 유형이다.\
리다이렉션 기반 플로우지만 액세스 토큰이 사용자 에이전트에 먼저 전달되기 때문에, 사용자나 사용자의 기기 내 다른 애플리케이션에 노출될 수 있다.\
애플리케이션의 ID를 인증하지 않으며 서비스에 등록한 리다이렉트 URI에 의존한다. 또한 새로고침 토큰을 지원하지 않는다.\
사용자는 애플리케이션 권한을 승인하도록 요청받고, 인증 서버는 액세스 토큰을 사용자 에이전트에 돌려주고,\
사용자 에이전트는 애플리케이션에게 그것을 패스하는 일련의 과정으로 수행된다.

#### Implicit 흐름

![](https://assets.digitalocean.com/articles/oauth/implicit_flow.png)

#### 1단계: 링크 제공

```
https://abc.com/v1/oauth/authorize?response_type=token&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
```

사용자에게 제공되는, API에 토큰을 요청하는 링크 예시이다.\
코드 대신 토큰을 요청한다는 것만 빼면 Authorization code 링크와 거의 똑같다. (response_type=token)

#### 2단계: 사용자는 애플리케이션에 권한을 승인한다

Authorization Code의 2단계와 동일하다.

#### 3단계: 사용자 에이전트는 리다이렉트 URI와 함께 액세스 토큰을 받는다

사용자가 애플리케이션을 승인하면, 서비스는 사용자 에이전트를 Redirect URI로 리다이렉트 시킨다.\
이 URI 프레그먼트에는 액세스 토큰이 포함된다.

```
https://dropletbook.com/callback#token=ACCESS_TOKEN
```

#### 4단계: 사용자 에이전트는 리다이렉트 URI를 따라간다

액세스 토큰을 간직한 채 리다이렉트 URI로 이동한다

#### 5단계: 애플리케이션이 액세스 토큰 추출 스크립트를 전달한다

애플리케이션은 리다이렉트 URI로부터 사용자 에이전트가 간직한 액세스 토큰을 추출할 수 있는 스크립트를 포함하는 웹페이지를 반환한다.

#### 6단계: 액세스 토큰이 애플리케이션에 전달된다

사용자 에이전트는 제공받은 스크립트를 수행하고, 추출된 액세스 토큰은 애플리케이션에 전달된다.\
이제 인증 절차가 완료되었다.\
토큰은 만료/취소되기 전까지 접근 수준 내에서 서비스 API를 통해 사용자의 계정에 접근할 수 있는 토큰으로 사용된다.

### Grant 유형 3. Resource Owner Password Credentials

사용자가 신뢰하는 애플리케이션에 적합한 유형이다. 이 유형을 사용할 때 인증 서버는 각별한 주의를 기울여야 한다.\
사용자의 credentials(username과 password)을 얻을 수 있는 애플리케이션의 경우에 사용할 수 있다.

#### Resource Owner Password Credentials 흐름
1. 사용자가 애플리케이션에 username과 password를 제공한다.
2. 애플리케이션이 사용자의 credentials를 갖고 인증 서버의 endpoint로부터 액세스 토큰을 요청한다.
3. 인증 서버는 애플리케이션을 인증하고 사용자 credentials를 검증한다. 통과되면 액세스 토큰이 발급된다.

### Grant 유형 4. Client Credentials

애플리케이션에 자체 서비스 계정에 접근하는 방법을 제공한다.\
애플리케이션이 등록된 부가설명이나 리다이렉트 URI를 갱신하기를 원하거나,\
API를 통해 서비스 계정에 저장된 다른 데이터에 접근하고자 원하는 경우에 유용하게 쓰인다.

#### Client Credentials 흐름

1. 애플리케이션은 그들의 credentials, 클라이언트 ID, 클라이언트 secret을 인증 서버에 전송함으로써 액세스 토큰을 요청한다.
2. 애플리케이션 credentials가 확인되면 인증 서버는 액세스 토큰을 애플리케이션에 돌려준다.

### 참고
- [An Introduction to OAuth 2](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2)
- [OAuth2 Overview](https://www.soapui.org/docs/oauth2/oauth2-overview/)
- [The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749)