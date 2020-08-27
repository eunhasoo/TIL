# 해싱

#### 해싱 _Hashing_

- 암호화 해시 함수로 알려진 수학적 함수를 이용해\
주어진 문자열으로부터 해시 또는 문자열을 생성해내는 과정
- 빠른 알고리즘은 해커의 무차별 대입 _brute force_ 공격에 취약하기 때문에\
암호 해시 함수의 처리 속도는 느린 것이 좋음
- 아래의 4가지 속성을 모두 갖는 해시 함수는 암호를 리버스 엔지니어링하는데 있어서\
어려움을 극도로 증가시키기 때문에 암호 해시 함수를 위한 강력한 후보가 될 수 있음

#### 보안 유지를 위한 4가지 속성

1. **결정론적이어야 한다.**\
같은 해시 함수로 처리된 동일한 메시지는 항상 동일한 해시를 생성한다.
2. **역으로 되돌릴 수 없다.**\
해시로부터 메시지를 생성해낼 수 없다.
3. **높은 엔트로피 _entropy_ 를 가진다.**\
메시지가 조금만 변해도 큰 차이를 갖는 해시를 생성해야한다.
4. **충돌에 저항한다.**\
두 개의 다른 메시지가 같은 해시를 생성해서는 안된다.

#### salt

- 데이터, 암호 또는 암호문를 해시하는 단방향 함수에 대한 추가 입력으로 사용되는 임의 데이터
- 암호 해싱에서 salt를 사용하는 것이 기본 원칙으로 자리잡음
- 각각의 새로운 해시마다 무작위로 생성되기 때문에 일반적으로 사용되는 비밀번호나\
여러 사이트에서 동일한 비밀번호를 사용하는 유저를 보호할 수 있음
- 레인보우 테이블과 같은 사전에 계산된 해시 공격에 대해 방어할 수 있음
- <참고> [salt 위키피디아](https://en.wikipedia.org/wiki/Salt_(cryptography))

### 해시 함수

#### 사용이 더 이상 비권장되는 함수들

- **MD5**
  - message-digest algorithm 사용
  - 충돌이 쉽게 발생할 수 있음
  - 속도가 빨라 무차별 대입 공격에 매우 취약

- **SHA-512**
  - 3세대 알고리즘에서 가장 긴 키로 대두됨
  - salt를 통해 해시의 엔트로피를 증가시키고\
사전 컴파일된 해시 목록으로부터 데이터베이스를 보호할 수 있음
  - 더 안전하고 느린 다른 옵션들이 존재하므로 비권장됨

#### 사용이 권장되는 함수들

- [PBKDF2](https://en.wikipedia.org/wiki/PBKDF2), [BCrypt](https://en.wikipedia.org/wiki/Bcrypt), [SCrypt](https://en.wikipedia.org/wiki/Scrypt), [Argon2](https://en.wikipedia.org/wiki/Argon2)
- 이들은 속도가 느리고, 강도를 설정할 수 있다는 중요한 특징이 있음\
즉, 입력을 변경함으로써 알고리즘의 속도를 늦출 수 있음

### Java를 이용한 구현

#### salt

```java
// SecureRandom 클래스를 사용해 salt를 생성한다.
SecureRandom random = new SecureRandom();
byte[] salt = new byte[16];
random.nextBytes(salt);
```

#### SHA-512

```java
// MessageDigest 클래스를 사용해 salt와 함께 SHA-512 해시 함수를 설정한다.
MessageDigest md = MessageDigest.getInstance("SHA-512");
md.update(salt);

// 해싱된 암호를 생성한다.
byte[] hashedPassword = md.digest(passwordToHash.getBytes(StandardCharsets.UTF_8));
```

#### PBKDF2

```java
//  PBEKeySpec와 PBKDF2 알고리즘을 사용해 인스턴스화시킬 SecretKeyFactory을 생성한다.
KeySpec spec = new PBEKeySpec(password.toCharArray(), salt, 65536, 128); 
// 세번째 파라미터인 65536은 얼마나 많이 알고리즘을 반복해서 수행할지 강도를 지정한 것
SecretKeyFactory factory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");

// SecretKeyFactory를 사용해 해시를 생성한다.
byte[] hash = factory.generateSecret(spec).getEncoded();
```

### Spring Security에서 제공하는 암호 해싱

- Java에서 PBKDF2와 SHA 해싱 알고리즘은 지원하지만,\
나머지 알고리즘들은 기본으로 제공되지 않는다
- Spring Security는 `PasswordEncoder` 인터페이스를 통해 모든 권장되는 알고리즘들을 지원한다
  - `MessageDigestPasswordEncoder` : MD5, SHA-512
  - `Pbkdf2PasswordEncoder` : PBKDF2
  - `BCryptPasswordEncoder` : BCrypt
  - `SCryptPasswordEncoder` : SCrypt

### 참고

- [Hashing a password in Java](https://www.baeldung.com/java-password-hashing)
- [PasswordEncoder](https://docs.spring.io/spring-security/site/docs/3.1.x/apidocs/org/springframework/security/crypto/password/PasswordEncoder.html)