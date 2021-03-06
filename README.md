# jwt 내용 정리

> jwt : JSON 객체를 사용해서 토큰 자체에 정보들을 저장하고 있는 Web Token이라고 정의

- JWT를 이용하는 방식은 헤비하지 않고 아주 간편하고 쉽게 적용할 수 있습니다.
- JWT는 Header, Payload, Signature 3개의 부분으로 구성되어 있습니다.

### Header

> Signature를 해싱하기 위한 알고리즘 정보가 담겨 있음

### Payload

> Payload는 서버와 클라이언트가 주고받는, 시스템에서 실제로 사용될 정보에 대한 내용들을 담고 있습니다.

```
{
  "sub": "1",
  "iss": "ori",
  "exp": 1636989718,
  "iat": 1636987918
}
```

- payload에는 보통 Claim이라는 사용자에 대한, 혹은 토큰에 대한 property를 key-value의 형태로 저장

- iss (Issuer) : 토큰 발급자
- sub (Subject) : 토큰 제목 - 토큰에서 사용자에 대한 식별값이 됨
- aud (Audience) : 토큰 대상자
- exp (Expiration Time) : 토큰 만료 시간
- nbf (Not Before) : 토큰 활성 날짜 (이 날짜 이전의 토큰은 활성화 되지 않음을 보장)
- iat (Issued At) : 토큰 발급 시간
- jti (JWT Id) : JWT 토큰 식별자 (issuer가 여러명일 때 이를 구분하기 위한 값)

### Signature

> 토큰의 유효성 검증을 위한 문자열

- 이 문자열을 통해 서버에서는 토큰이 유효한 토큰인지 검증할 수 있습니다.

### :book: JWT 장점

- 중앙의 인증서버, 데이터 스터어에 대한 의존성이 없고, 시스템 수평 확장에 유리합니다.
- Base64 URL Safe Encoding > URL, Cookie, Header 모두 사용이 가능합니다.

### :book: JWT 단점

- Payload의 정보가 많아지면 네트워크 사용량이 증가하며, 데이터 설계를 고려할 필요가 있습니다.
- 토큰이 클라이언트에 저장, 서버에서 클라이언트의 토큰을 조작할 수 없습니다.

## Code 설명 정리

### 환경설정

```
jwt:
  header: Authorization
  secret: c2lsdmVybmluZS10ZWNoLXNwcmluZy1ib290LWp3dC10dXRvcmlhbC1zZWNyZXQtc2lsdmVybmluZS10ZWNoLXNwcmluZy1ib290LWp3dC10dXRvcmlhbC1zZWNyZXQK
  token-validity-in-seconds: 86400
```

- HS512 알고리즘을 사용할 것이기 때문에 512bit, 즉 64byte 이상의 secret key를 사용해야 한다.
- echo 'silvernine-tech-spring-boot-jwt-tutorial-secret-silvernine-tech-spring-boot-jwt-tutorial-secret'|base64

### TokenProvider

- InitializingBean을 implements해서 afterPropertiesSet을 Override한 이유는 빈이 생성이 되고 주입을 받은 후에 secret값을 Base64 Decode해서 key 변수에 할당하기 위해서

- Authentication 객체의 권한정보를 이용해서 토큰을 생성하는 createToken 메소드 추가

- Token에 담겨있는 정보를 이용해 Authentication 객체를 리턴하는 메소드 생성

- validateToken 메소드 -> 토큰의 유효성 검사

### JwtFilter

- GenericFilterBean을 extends 해서 doFilter Override, 실제 필터링 로직은 doFilter 내부에 작성

- doFilter는 토큰의 인증정보를 SecurityContext에 저장하는 역활 수행

- RequestHeader에서 토큰 정보를 꺼내오기 위한 resolveToken 메소드 추가

### CreateToken

- signWith : 위에서 이야기한 개인키를 가지고 HS512 암호화 알고리즘으로 header와 payload로 Signature를 생성.
- setHeaderParam : header에 typ로 JWT 지정
- setSubject : 이후 JWT로 인증할 식별자를 user 객체의 id로 지정
- setIssuer : JWT 발급자 지정
- setExpiration : 현재 시간으로부터 지정된 accessTime만큼의 만료시간 지정
- setIssueAt : JWT를 발급한 시간 지정
