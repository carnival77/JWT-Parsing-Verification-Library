# JWT-Helper-Library

## 개요

### 설명

JWT(Json Web Token)를 생성하고, 파싱하고, 서명을 검증하며, 토큰에 포함된 데이터를 활용할 수 있는 간단한 스프링 기반 유틸리티 라이브러리입니다. Spring 기반 애플리케이션에서 JWT 기반 인증 흐름을 지원하여 애플리케이션 전반의 인증 로직을 단순화하고 보안성을 강화합니다.

### 주요 기능

- JWT 토큰 파싱
- JWT 유효성 검증
- Claim 데이터 추출 및 활용
- Secret Key 및 설정 관리
- JWT 생성
- 예외 처리

### 사용 설명

#### 라이브러리 사용 방법

**Gradle-Kotlin 설정**

- `settings.gradle.kts`

  ```yaml
  dependencyResolutionManagement {
      repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
      repositories {
          mavenCentral()
          maven {
              url = uri("https://jitpack.io")
          }
      }
  }
  ```

- `build.gradle.kts`

  ```yaml
  dependencies {
      implementation("com.github.carnival77:JWT-Helper-Library:1.0.1")
  }
  ```

#### YAML 설정

JWT 설정은 YAML 파일을 통해 관리됩니다.

**예시 YAML:**

```yaml
jwt:
  cases:
    create-account:
      secretKey: your-256-bit-secret-your-256-bit-secret-your-256-bit-secret1
      expiration: 1200000 # 20 min
      algorithm: HS256
      claims: ['email', 'name']
    login:
      secretKey: your-256-bit-secret-your-256-bit-secret-your-256-bit-secret3
      expiration: 3600000 # 1 hour
      algorithm: RS512
      claims: ['role']
```

#### JWT 생성 (`JWTGenerator`)

- 사용자 정의 `Claims`와 필수 `Claims`를 기반으로 JWT 토큰을 생성합니다.
- YAML 설정(`JWTProperties`)에 정의된 사용 사례별(secretKey, expiration 등) 구성을 사용합니다.
- 추가 `Claims`를 병합하여 동적으로 확장 가능합니다.

**예시 코드:**

```java
String token = jwtGenerator.generateToken(
    "login",                // Use Case
    "user1",                // Subject
    Map.of("role", "USER"), // Claims
    Map.of("email", "test@example.com") // 추가 Claims
);
```

#### JWT 검증 (`JWTService`)

- JWT 토큰의 유효성을 검사하고, 토큰의 `Claims`를 반환합니다.
- 토큰의 만료, 누락된 필수 `Claims`, 서명 불일치 등 다양한 예외를 처리합니다.

**예시 코드:**

```java
try {
    Claims claims = jwtService.validateAndClaims(token, "create-account");
    System.out.println("Subject: " + claims.getSubject());
} catch (TokenExpiredException e) {
    System.out.println("Token has expired.");
} catch (MissingClaimsException e) {
    System.out.println("Missing claims: " + e.getMessage());
}
```

#### Claims` 추출 (`ClaimsExtractor`)

- 검증된 `Claims` 객체에서 데이터를 추출합니다.
- `subject`, `role`, `email` 등의 데이터를 간편하게 가져올 수 있습니다.

**예시 코드:**

```java
String subject = ClaimsExtractor.getSubject(validatedClaims);
String email = ClaimsExtractor.getStringClaims(validatedClaims, "email");
```

#### 예외 처리

| 예외 클래스              | 설명                                                        |
| ------------------------ | ----------------------------------------------------------- |
| `InvalidTokenException`  | 유효하지 않은 토큰 (서명 불일치, 구조적 문제) 발생 시 사용. |
| `TokenExpiredException`  | 토큰이 만료되었을 때 발생.                                  |
| `MissingClaimsException` | 필수 `Claims`가 누락되었을 때 발생.                         |

## 라이브러리의 역할 및 가치



1. 인증/인가 로직 전개를 위한 기초 제공

   - 다른 인증 수단(로그인 서비스, OAuth 서버 등)에서 발급한 JWT 토큰을 검증하고, 유효한 토큰에서 사용자 식별 정보를 추출하는 역할을 한다.
   - 이로써 애플리케이션 내에서 인증 흐름을 확립하는데 핵심적인 토대를 제공한다.

2. 표준 기반의 확장성

   - JWT는 널리 사용되는 표준(RFC 7519)이므로, 이 라이브러리를 활용하면 다양한 언어, 서비스, 게이트웨이, OAuth 인증 서버 등과 쉽게 연계할 수 있다.
   - 필요할 경우 RSA, EC 등 다양한 알고리즘 지원, Blacklist 토큰 테이블 연계, Refresh 토큰 처리, OAuth 인증 서버와의 연동 등 확장 가능하다.

3. 구현 단순화

   - 애플리케이션 개발자가 매번 토큰 파싱과 검증 로직을 직접 구현할 필요 없이, 해당 라이브러리의 간단한 인터페이스(API) 호출로 토큰 유효성 검사 및 Claim 접근이 가능하다.
   - 이를 통해 중복 코드 감소와 유지보수성 향상을 기대할 수 있다.

4. 인증(Authentication)

   - JWT를 통해 사용자 인증
     - 사용자가 로그인하면 서버에서 JWT를 생성하여 클라이언트에 전달.
     - 클라이언트는 JWT를 헤더(예: `Authorization: Bearer <토큰>`)에 포함해 요청.
     - 서버는 JWT를 파싱하여 사용자의 ID를 인증.

5. 권한 부여(Authorization)

   - 사용자 권한에 따라 API 접근 제어
     - JWT의 Claim 데이터(`role`, `permissions` 등)를 사용해 요청자의 권한 확인.
     - 예: `role=ADMIN`이면 관리자 API 접근 허용, `role=USER`이면 제한.

6. 보안(Security)

   - 안전한 요청 처리
     - 서명(Signature) 검증으로 JWT가 위조되지 않았는지 확인.
     - 만료(exp) 검증으로 사용 기한이 지난 JWT를 차단.
     - Secret Key를 활용하여 암호화된 서명 생성 및 검증.

7. 간편한 토큰 관리

   - 유연한 JWT 생성 및 관리
     - 사용자 정보를 담아 필요한 정보를 JWT로 전달.
     - Secret Key와 만료 시간을 설정해 보안 강화.
     - 토큰 기반 세션 관리로 상태 비저장(Stateless) 인증 구현.

## **사용 사례**

1. 사용자 로그인
   - 사용자가 로그인 시 JWT를 생성하여 반환.
   - 클라이언트는 이후 요청 시 JWT를 헤더에 포함해 전송.
2. API 접근 제어
   - 관리자만 접근 가능한 API에서 JWT의 Claim(`role`)을 확인.
   - 특정 사용자 그룹만 접근 가능한 API에서 Claim(`group`)을 기반으로 검증.
3. 마이크로서비스 인증
   - 서비스 간 통신 시 JWT를 사용하여 호출하는 서비스의 유효성을 검증.

## **확장 가능성**



- **Refresh Token 관리**: 만료된 토큰을 갱신하는 로직 추가.
- **API Gateway 통합**: JWT 검증을 Gateway에서 처리하도록 설정.
- **Custom Claim 지원**: 사용자 지정 데이터(예: `department`, `project_id`)를 추가하여 권한 부여 로직 확장.
- **OAuth 또는 SSO 연동:** OAuth 인증 과정에서 JWT를 활용하여 클라이언트와 서버 간 인증 토큰으로 사용.
