# 4단계: 실전 기능 추가 (Advanced Features)

## 1. Spring Security

Spring Security는 Spring 기반 애플리케이션의 **보안(인증 및 권한 부여)**을 담당하는 강력한 프레임워크입니다. 웹 애플리케이션에서 발생할 수 있는 다양한 보안 위협으로부터 시스템을 보호하기 위한 포괄적인 기능들을 제공합니다. 개발자가 보안 관련 로직을 직접 작성하는 수고를 덜어주고, 보다 쉽고 체계적으로 보안 기능을 구현할 수 있도록 도와줍니다.

---

### **핵심 개념**

Spring Security를 이해하기 위해서는 두 가지 핵심 개념을 반드시 알아야 합니다.

#### **1) 인증 (Authentication)**

- **의미:** 사용자가 누구인지 **신원을 확인**하는 과정입니다. 간단히 말해 '로그인'과 같은 절차를 의미합니다.
- **과정:** 사용자가 아이디와 비밀번호 같은 자격 증명(Credentials)을 제출하면, 시스템은 이 정보가 유효한지 검증하여 사용자를 식별합니다. 인증이 성공하면, 해당 사용자의 정보와 권한을 담은 `Authentication` 객체가 생성되어 `SecurityContext`에 저장됩니다.

#### **2) 인가 (Authorization)**

- **의미:** 인증된 사용자가 특정 리소스(Resource)나 기능에 **접근할 수 있는 권한이 있는지 확인**하는 과정입니다.
- **과정:** 사용자가 특정 URL(예: `/admin/dashboard`)에 접근을 시도할 때, Spring Security는 `SecurityContext`에 저장된 `Authentication` 객체를 확인하여 해당 사용자가 이 리소스에 접근하는 데 필요한 권한(예: `ROLE_ADMIN`)을 가지고 있는지 검사합니다.

---

### **주요 구성 요소와 동작 원리**

Spring Security는 **서블릿 필터(Servlet Filter)** 체인을 기반으로 동작합니다. 클라이언트의 요청이 DispatcherServlet에 도달하기 전에 여러 보안 필터들을 거치면서 인증 및 인가 작업이 수행됩니다.

- **`SecurityFilterChain`**: 보안을 위해 적용해야 할 필터들의 체인입니다. 개발자는 이 체인을 커스터마이징하여 필요한 보안 설정을 구성할 수 있습니다. (예: 특정 URL 경로는 인증 없이 허용, 특정 경로는 특정 권한 필요 등)
- **`AuthenticationManager`**: 인증 요청을 처리하는 인터페이스입니다. 실제 인증은 `AuthenticationManager`에 등록된 `AuthenticationProvider`가 수행합니다.
- **`UserDetailsService`**: 데이터베이스나 다른 저장소에서 사용자 정보를 불러오는 역할을 합니다. 사용자의 아이디를 기반으로 `UserDetails` 객체를 반환합니다.
- **`PasswordEncoder`**: 사용자의 비밀번호를 안전하게 저장하고 검증하기 위해 비밀번호를 암호화하는 역할을 합니다. `BCryptPasswordEncoder`가 일반적으로 사용됩니다.

---

### **기본 설정 코드 예시 (Configuration Example)**

Spring Security의 설정은 주로 `@Configuration` 어노테이션을 붙인 클래스에서 `SecurityFilterChain` 타입의 Bean을 등록하여 정의합니다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity // Spring Security 활성화
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        // 비밀번호 암호화를 위한 PasswordEncoder 빈 등록 (BCrypt 사용)
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // CSRF(Cross-Site Request Forgery) 보호 비활성화 (Stateless API에서는 보통 비활성화)
            .csrf(csrf -> csrf.disable())

            // 세션 관리 정책 설정: STATELESS (상태 비저장)
            // JWT 기반 인증을 사용하므로 세션을 사용하지 않음
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            // HTTP 요청에 대한 인가 규칙 설정
            .authorizeHttpRequests(authorize -> authorize
                // 아래 경로에 대한 요청은 인증 없이 허용
                .requestMatchers("/api/auth/**", "/swagger-ui/**", "/v3/api-docs/**").permitAll()
                // 'ADMIN' 역할을 가진 사용자만 접근 허용
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                // 그 외 모든 요청은 인증된 사용자만 접근 가능
                .anyRequest().authenticated()
            );
            
            // (추후 추가될 부분) JWT 인증 필터를 UsernamePasswordAuthenticationFilter 앞에 추가
            // .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

#### **코드 설명:**

1.  **`@Configuration` & `@EnableWebSecurity`**: 이 클래스가 Spring의 설정 파일임을 알리고, Spring Security를 활성화합니다.
2.  **`passwordEncoder()` Bean**:
    -   `PasswordEncoder`는 비밀번호를 안전하게 암호화하고, 로그인 시 제출된 비밀번호와 DB에 저장된 암호화된 비밀번호를 비교하는 데 사용됩니다.
    -   `BCryptPasswordEncoder`는 현재 가장 널리 사용되는 안전한 해싱 알고리즘 중 하나입니다.
3.  **`securityFilterChain(HttpSecurity http)` Bean**:
    -   이 메서드가 Spring Security의 핵심 설정 부분입니다. `HttpSecurity` 객체를 사용하여 웹 기반 보안을 구성합니다.
    -   **`.csrf(csrf -> csrf.disable())`**: REST API 서버는 보통 세션 대신 토큰을 사용하므로, 세션 기반의 CSRF 공격에 비교적 안전합니다. 따라서 CSRF 보호 기능을 비활성화합니다.
    -   **`.sessionManagement(...)`**: 세션 정책을 `STATELESS`로 설정합니다. 이는 서버가 클라이언트의 상태를 저장하지 않겠다는 의미로, JWT와 같은 토큰 기반 인증에 필수적인 설정입니다.
    -   **`.authorizeHttpRequests(...)`**: HTTP 요청에 대한 접근 권한을 설정합니다.
        -   `requestMatchers(...).permitAll()`: `/api/auth/**` (회원가입, 로그인 등), `/swagger-ui/**` (API 문서) 같은 특정 경로는 누구나 접근할 수 있도록 허용합니다.
        -   `requestMatchers(...).hasRole("ADMIN")`: `/api/admin/**` 경로는 `ADMIN` 역할을 가진 사용자만 접근할 수 있습니다.
        -   `anyRequest().authenticated()`: 위에서 지정한 경로 외의 모든 요청은 반드시 인증(로그인)을 거쳐야만 접근할 수 있습니다.
    -   **`.addFilterBefore(...)`**: (주석 처리된 부분) 나중에 JWT 인증을 구현할 때, 직접 만든 `JwtAuthenticationFilter`를 Spring Security의 기본 필터 체인에 추가하는 부분입니다.

---

### **JWT (JSON Web Token)를 이용한 인증**

현대적인 RESTful API 서버에서는 상태를 저장하지 않는(Stateless) 인증 방식인 JWT가 널리 사용됩니다.

- **동작 방식:**
    1.  **로그인:** 사용자가 아이디/비밀번호로 로그인을 요청합니다.
    2.  **토큰 발급:** 서버는 사용자 인증에 성공하면, 사용자의 정보와 권한을 담은 JWT를 생성하여 암호화한 후 클라이언트에게 발급합니다.
    3.  **요청 시 토큰 사용:** 클라이언트는 발급받은 JWT를 저장해두고, 이후 서버에 API를 요청할 때마다 HTTP 헤더(보통 `Authorization: Bearer <token>`)에 토큰을 포함하여 보냅니다.
    4.  **토큰 검증:** 서버는 요청 헤더에 담긴 JWT를 검증하여 유효한 토큰인지 확인하고, 토큰에 포함된 사용자 정보를 바탕으로 인가 처리를 수행합니다.

- **장점:**
    - **Stateless:** 서버가 클라이언트의 세션 상태를 저장할 필요가 없어 서버 확장성(Scale-out)에 유리합니다.
    - **자가 수용적(Self-contained):** 토큰 자체에 필요한 모든 정보(사용자 식별 정보, 권한 등)를 담고 있습니다.
    - **유연성:** 웹, 모바일 등 다양한 클라이언트 환경에서 쉽게 사용될 수 있습니다.

---

### **요약**

- **Spring Security**는 Spring 애플리케이션의 **인증(Authentication)**과 **인가(Authorization)**를 전문적으로 처리하는 강력한 보안 프레임워크입니다.
- **인증**은 사용자가 누구인지 확인하는 과정(로그인)이고, **인가**는 인증된 사용자가 리소스에 접근할 권한이 있는지 확인하는 과정입니다.
- 서블릿 필터 체인을 기반으로 동작하며, 개발자는 `SecurityFilterChain`을 설정하여 보안 규칙을 커스터마이징할 수 있습니다.
- 현대 API 서버에서는 상태를 저장하지 않는 **JWT** 방식이 표준처럼 사용되며, 서버 확장성과 유연성이 뛰어납니다.

---

### **출처 및 참고 자료**

- **공식 문서:**
    - Spring Security 공식 문서: [https://spring.io/projects/spring-security](https://spring.io/projects/spring-security)
    - Spring 공식 가이드 - Securing a Web Application: [https://spring.io/guides/gs/securing-web/](https://spring.io/guides/gs/securing-web/)
- **개념 이해를 위한 블로그:**
    - [SpringBoot] Spring Security란? (망나니개발자): [https://mangkyu.tistory.com/78](https://mangkyu.tistory.com/78)
    - Spring Security 개념과 처리 과정 (velog): [https://velog.io/@h-log/Spring-Spring-Security-개념과-처리-과정](https://velog.io/@h-log/Spring-Spring-Security-개념과-처리-과정)
- **JWT 관련 자료:**
    - [JWT] JSON Web Token 소개 및 구조 (velopert): [https://velopert.com/2389](https://velopert.com/2389)