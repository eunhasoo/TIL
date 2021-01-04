

# OAuth 2

### 간단한 정의

- 페이스북, 깃허브와 같은 HTTP 서비스의 사용자 계정에 제한적으로 접근할 수 있도록 하는 인증 _authorization_ 프레임워크
- 사용자 계정을 호스팅하는 서비스에 사용자 인증을 위임하고, 제3자 애플리케이션이 사용자 계정에 접근할 수 있도록 권한을 부여

### 역할

#### 1. 리소스 소유자: 사용자

리소스 소유자 _Resource Owner_ 는 자신의 계정에 접근하고자 애플리케이션에 인증을 승인받으려는 사용자들이다.\
사용자 계정에 대한 애플리케이션의 접근은 부여된 권한의 범위에 제한적이다. (ex. 읽기/쓰기 권한)

#### 2.리소스/인증 서버: API

리소스 서버 _Resource Server_ 는 보호된 사용자 계정을 호스팅하고, 
인증 서버 _Authorization Server_ 는 사용자의 ID를 확인하여 애플리케이션에 액세스 토큰 _access token_ 을 발행한다.
애플리케이션 개발자의 관점에서, 서비스의 API는 리소스와 인증 서버의 두 역할을 모두 수행한다.
> 두 역할을 결합해 서비스 혹은 API 역할이라고도 한다.

#### 3. 클라이언트: 애플리케이션

클라이언트는 사용자 계정에 접근을 원하는 애플리케이션이다.\
계정에 접근하기 이전에 사용자에 의해 권한이 부여되어야 하며, API를 통해 권한이 검증되어야 한다.

### 추상적인 프로토콜의 흐름

![](https://assets.digitalocean.com/articles/oauth/abstract_flow.png)

1. _애플리케이션_ 은 _사용자_ 에게 서비스 리소스에 접근할 수 있는 권한을 요청한다.
2. _사용자_ 가 요청에 대해 인증했다면, _애플리케이션_ 은 권한을 부여받는다.
3. _애플리케이션_ 은 자체 ID 인증 및 권한 부여를 제공하며 _인증 서버_ (API)에 액세스 토큰을 요청한다.
4. 분명한 애플리케이션 ID이면서 권한 부여가 유효한 경우, _인증 서버_ 는 애플리케이션에 액세스 토큰을 발행한다. 인증 절차가 완료된다.
5. _애플리케이션_ 은 _리소스 서버_ (API)에 리소스를 요청하고 인증을 위한 액세스 토큰을 제공한다.
6. 액세스 토큰이 유효하면, _리소스 서버_ 는 _애플리케이션_ 에 리소스를 제공한다.

> 이 과정에서 실제 흐름은 사용되는 권한 부여 타입에 따라 다를 수 있다.

### 애플리케이션 등록

OAuth 사용하기 전에 서비스에 내 애플리케이션을 등록하는 절차가 필요한데,\
서비스 웹사이트 내 개발자 혹은 API에 관련된 부분에서 이를 수행할 수 있다.\
`애플리케이션 이름`, `웹사이트`, `리다이렉트 URI / 콜백 URL`과 같은 애플리케이션 정보를 제공해야 할 것이다.\
리다이렉트 URI는 인증을 수행한 (혹은 거부당한) 사용자들을 리다이렉트해줄 위치이므로,\
애플리케이션에서 액세스 토큰이나 권한 부여 코드를 처리하는 부분이다.

#### Client Identifier, Client Secret

애플리케이션이 등록되고 나면, 서비스는 Client Identifier와 Client Secret의 형태로 Client Credentials을 발행해준다.\
Client Identifier는 서비스 API에서 애플리케이션을 식별하기 위해 공개적으로 노출되는 문자열이며, 사용자에게 제공되는 인증 URL을 빌드하는 데에도 사용된다.\
Client Secret은 애플리케이션이 사용자의 계정에 대한 접근을 서비스 API에게 요청했을 때 애플리케이션의 식별을 증명하는 데에 사용되고, API와 애플리케이션 사이에서 프라이빗하게 유지되어야 한다.

### 권한 부여 Authorization Grant

위 추상적인 프로토콜 흐름 부분에서, 1-4번은 권한 부여와 액세스 토큰을 얻는 단계이다.\
권한 부여 유형은 애플리케이션이 권한을 요청하기 위해 사용되는 방법과 API에서 지원되는 부여 타입에 따라 달라진다.

- `Authorization Code`: 서버측 애플리케이션에서 사용
- `Implicit`: 모바일 앱 또는 (사용자의 기기에서 실행되는) 웹 애플리케이션에서 사용
- `Resource Owner Password Credentials`: 서비스 자체 소유이거나 신뢰할 수 있는 애플리케이션에서 사용
- `Client Credentials`: 애플리케이션 API 액세스에 사용

### Grant 유형 1. Autorization Code

**가장 일반적으로 사용되는** 부여 유형이다.\
소스 코드가 공개적으로 노출되지 않고, Client Secret의 기밀성이 유지될 수 있는 서버 측 애플리케이션에 최적이기 때문이다.\
이는 **리다이렉션 기반** 플로우이다. 즉, 애플리케이션이 웹 브라우저같은 사용자 에이전트와 상호작용하거나\
사용자 에이전트를 통해 라우팅되는 API 인증 코드를 수신할 수 있음을 의미한다.

#### Autorization Code 흐름

![](https://assets.digitalocean.com/articles/oauth/auth_code_flow.png)

#### 1단계: 인증 코드 링크

```url
https://abc.com/v1/oauth/authorize?response_type=code&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
```

사용자에게 이러한 인증 코드가 부여될 때 링크의 컴포넌트를 살펴보면

- `https://abc.com/oauth/authorize`: API 인증 endpoint
- `client_id=client_id`: 애플리케이션의 클라이언트 ID (API가 애플리케이션을 식별하는 방법)
- `response_type=code`: 애플리케이션이 인증 코드 부여를 요청중임을 명시
- `scope=read`: 애플리케이션이 요청중인 접근의 레벨을 명시

#### 2단계: 사용자가 애플리케이션을 인증한다

사용자가 링크를 클릭하면 그들은 ID 인증을 위해 먼저 서비스에 로그인을 해야 한다.\
이후 서비스는 그들의 계정에 대한 애플리케이션 접근을 승인하거나 거부할 수 있는 메시지를 표시한다.

#### 3단계: 애플리케이션이 인증 코드를 받는다.

사용자가 애플리케이션에 권한을 승인하면, 서비스는 인증 코드가 달린 애플리케이션 리다이렉트 URI으로 사용자 에이전트를 이동시킨다.\
(이 리다이렉트 주소는 클라이언트 등록 절차에서 명시한 URI이다)

#### 4단계: 애플리케이션이 액세스 토큰을 요청한다

애플리케이션은 client secret을 포함한 인증 세부 정보와 인증 코드를 넘기면서 API에 액세스 토큰을 요청한다.

```
https://cloud.digitalocean.com/v1/oauth/token?client_id=CLIENT_ID&client_secret=CLIENT_SECRET&grant_type=authorization_code&code=AUTHORIZATION_CODE&redirect_uri=CALLBACK_URL
```

#### 5단계: 애플리케이션이 액세스 토큰을 받는다

인증이 유효하다면 API는 아래와 같은 액세스 토큰을 포함한 응답을 애플리케이션에 보낼 것이다.

```
{"access_token":"ACCESS_TOKEN","token_type":"bearer","expires_in":2592000,"refresh_token":"REFRESH_TOKEN","scope":"read","uid":100101,"info":{"name":"Mark E. Mark","email":"mark@thefunkybunch.com"}}
```

이제 애플리케이션은 인증되었다.\
만료 혹은 취소되지 않는 한, 서비스 API를 통해 접근 범위 안에서 사용자 계정에 접근하기 위한 토큰으로 사용되어질 것이다.

### Grant 유형 2. Implicit

client secret 기밀성이 보장되지 않는 모바일 앱, 웹 애플리케이션(ex. 웹 브라우저에서 실행되는 애플리케이션)에서 사용되는 유형이다.\
리다이렉션 기반 플로우지만 액세스 토큰이 애플리케이션 전에 사용자 에이전트에 먼저 부여되기 때문에, 사용자나 사용자의 기기 내 다른 애플리케이션에 노출될 수 있다.\
애플리케이션의 ID를 인증하지 않으며 서비스에 등록한 리다이렉트 URI에 의존한다. 또한 새로고침 토큰을 지원하지 않는다.\
사용자는 애플리케이션 권한을 승인하도록 요청받고, 인증 서버는 액세스 토큰을 사용자 에이전트에 돌려주고,\
사용자 에이전트는 애플리케이션에게 그것을 패스하는 일련의 과정으로 수행된다.

#### Implicit 흐름

![](https://assets.digitalocean.com/articles/oauth/implicit_flow.png)

#### 1단계: Implicit 인증 링크

```
https://cloud.digitalocean.com/v1/oauth/authorize?response_type=token&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=read
```

사용자에게 제공되는, API에 토큰을 요청하는 권한 부여 링크이다.\
코드 대신 `토큰`을 요청한다는 것만 빼면 autorization code 링크와 거의 똑같다. (response_type=token)

#### 2단계: 사용자가 애플리케이션을 인증한다

(authorization code flow의 2단계와 동일)

#### 3단계: 사용자 에이전트는 리다이렉트 URI와 함께 액세스 토큰을 받는다

사용자가 애플리케이션을 승인하면, 서비스는 사용자 에이전트를 리다이렉트 URI로 이동시킨다.\
URI 조각에는 액세스 토큰이 포함된다.

#### 4단계: 사용자 에이전트가 리다이렉트 URI을 따라간다

액세스 토큰을 간직한채로 리다이렉트 URI로 이동한다

#### 5단계: 애플리케이션이 액세스 토큰 추출 스크립트를 전달한다

애플리케이션은 사용자 에이전트가 간직한 리다이렉트 URI로부터 액세스 토큰을 추출할 수 있는 스크립트를 포함하는 웹페이지를 돌려준다.

#### 6단계: 액세스 토큰이 애플리케이션에 전달된다

사용자 에이전트는 제공받은 스크립트를 수행하고 추출된 액세스 토큰을 애플리케이션에 보낸다.\
이제 애플리케이션이 인증되었다. 만료되거나 취소되기 전까지는 접근 범위 내에서 서비스 API를 통해 사용자의 계정에 접근할 수 있는 토큰으로 사용된다.

### Grant 유형 3. Resource Owner Password Credentials


### 출처
- [An Introduction to OAuth 2](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2)
- [OAuth2 Overview](https://www.soapui.org/docs/oauth2/oauth2-overview/)