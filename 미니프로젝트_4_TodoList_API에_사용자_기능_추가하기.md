### **미니 프로젝트 #4: To-Do List API에 사용자 기능 추가하기**

이전 미니 프로젝트 #3에서 만든 To-Do List API에 Spring Security와 JWT(JSON Web Token)를 적용하여 사용자 인증/인가 기능을 추가합니다. 이제 각 사용자는 회원가입 후 자신만의 To-Do 리스트를 관리할 수 있게 됩니다.

**핵심 목표:**
1.  회원가입 및 로그인 API 구현
2.  JWT를 이용한 토큰 기반 인증 시스템 구축
3.  사용자별로 To-Do 데이터가 격리되도록 API 보안 처리

---

#### **1단계: 의존성 추가 (`build.gradle`)**

기존 의존성에 Spring Security와 JWT 라이브러리를 추가합니다.

-   `spring-boot-starter-security`: Spring Boot 환경에서 보안 기능을 쉽게 설정할 수 있도록 도와주는 라이브러리
-   `jjwt-api`, `jjwt-impl`, `jjwt-jackson`: JWT를 생성하고 검증하기 위한 라이브러리

```groovy
// build.gradle
dependencies {
    // 기존 의존성
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'

    // --- 추가된 의존성 ---
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-validation' // 유효성 검사를 위해 추가
    implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
    runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
    // --------------------
}
```

#### **2단계: User 엔티티 및 Repository 만들기**

사용자 정보를 저장할 `User` 엔티티와 `UserRepository`를 만듭니다.

**경로:** `src/main/java/com/example/myfirstapp/user/User.java`
(user 패키지를 새로 만들어주세요.)
```java
package com.example.myfirstapp.user;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Entity
@Getter
@Setter
@NoArgsConstructor
@Table(name = "users") // "user"는 DB 예약어인 경우가 많아 "users" 사용
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(nullable = false)
    private String password;

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }
}
```

**경로:** `src/main/java/com/example/myfirstapp/user/UserRepository.java`
```java
package com.example.myfirstapp.user;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

#### **3단계: Todo 엔티티 수정**

`Todo`가 어떤 `User`에 속해있는지 알려주기 위해 관계를 맺어줍니다.

**경로:** `src/main/java/com/example/myfirstapp/todo/Todo.java`
```java
// ... 기존 import ...
import com.example.myfirstapp.user.User;
import jakarta.persistence.*; // ManyToOne, JoinColumn 추가

// ... 기존 어노테이션 ...
@Entity
public class Todo {
    // ... 기존 필드 ...
    private String title;
    private boolean completed;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    public void setUser(User user) {
        this.user = user;
    }
}
```

#### **4단계: JWT 및 Security 설정**

**1. JWT Util 클래스 만들기**
JWT 토큰의 생성, 검증, 정보 추출을 담당하는 헬퍼 클래스입니다.

**경로:** `src/main/java/com/example/myfirstapp/security/JwtUtil.java`
(security 패키지를 새로 만들어주세요.)
```java
package com.example.myfirstapp.security;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import jakarta.annotation.PostConstruct;
import jakarta.servlet.http.HttpServletRequest;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import java.security.Key;
import java.util.Base64;
import java.util.Date;

@Component
public class JwtUtil {

    public static final String AUTHORIZATION_HEADER = "Authorization";
    public static final String BEARER_PREFIX = "Bearer ";

    @Value("${jwt.secret.key}")
    private String secretKey;
    private Key key;
    private final SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;

    @PostConstruct
    public void init() {
        byte[] bytes = Base64.getDecoder().decode(secretKey);
        key = Keys.hmacShaKeyFor(bytes);
    }

    public String createToken(String username) {
        Date date = new Date();
        long TOKEN_TIME = 60 * 60 * 1000L; // 1시간

        return BEARER_PREFIX +
                Jwts.builder()
                        .setSubject(username)
                        .setExpiration(new Date(date.getTime() + TOKEN_TIME))
                        .setIssuedAt(date)
                        .signWith(key, signatureAlgorithm)
                        .compact();
    }

    public String getJwtFromHeader(HttpServletRequest request) {
        String bearerToken = request.getHeader(AUTHORIZATION_HEADER);
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith(BEARER_PREFIX)) {
            return bearerToken.substring(7);
        }
        return null;
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token);
            return true;
        } catch (Exception e) {
            // 로그 기록
        }
        return false;
    }

    public Claims getUserInfoFromToken(String token) {
        return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token).getBody();
    }
}
```

**2. `application.properties`에 JWT 비밀 키 추가**
아래 내용을 `application.properties` 파일에 추가하세요. 값은 충분히 길고 복잡한 문자열로 바꾸는 것이 좋습니다.
```properties
# JWT Secret Key
jwt.secret.key=7ZWt7ZW0Ode+7YWM7Iqk7Yq47J6F7ZWY7Y247ZWt7ZW0Ode+7YWM7Iqk7Yq47J6F7ZWY7Y24Cg==
```

**3. UserDetails 구현체 및 Service 만들기**
Spring Security가 사용자의 정보를 가져갈 때 필요한 클래스들입니다.

**경로:** `src/main/java/com/example/myfirstapp/security/UserDetailsImpl.java`
```java
package com.example.myfirstapp.security;

import com.example.myfirstapp.user.User;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.Collections;

public class UserDetailsImpl implements UserDetails {

    private final User user;

    public UserDetailsImpl(User user) {
        this.user = user;
    }

    public User getUser() {
        return user;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    // 계정 만료, 잠금, 자격 증명 만료, 활성화 여부 등은 모두 true로 간단히 처리
    @Override
    public boolean isAccountNonExpired() { return true; }
    @Override
    public boolean isAccountNonLocked() { return true; }
    @Override
    public boolean isCredentialsNonExpired() { return true; }
    @Override
    public boolean isEnabled() { return true; }

    // 권한은 현재 사용하지 않으므로 빈 리스트 반환
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.emptyList();
    }
}
```

**경로:** `src/main/java/com/example/myfirstapp/security/UserDetailsService.java`
```java
package com.example.myfirstapp.security;

import com.example.myfirstapp.user.User;
import com.example.myfirstapp.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserDetailsService {

    private final UserRepository userRepository;

    public UserDetailsImpl loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("Not Found " + username));

        return new UserDetailsImpl(user);
    }
}
```

**4. JWT 인증 필터 및 Security 설정 클래스 만들기**

**경로:** `src/main/java/com/example/myfirstapp/security/JwtAuthorizationFilter.java`
```java
package com.example.myfirstapp.security;

import io.jsonwebtoken.Claims;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@RequiredArgsConstructor
public class JwtAuthorizationFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String token = jwtUtil.getJwtFromHeader(request);

        if (token != null) {
            if (jwtUtil.validateToken(token)) {
                Claims info = jwtUtil.getUserInfoFromToken(token);
                String username = info.getSubject();
                SecurityContext context = SecurityContextHolder.createEmptyContext();
                UserDetailsImpl userDetails = userDetailsService.loadUserByUsername(username);
                Authentication authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                context.setAuthentication(authentication);
                SecurityContextHolder.setContext(context);
            }
        }
        filterChain.doFilter(request, response);
    }
}
```

**경로:** `src/main/java/com/example/myfirstapp/config/SecurityConfig.java`
(config 패키지를 새로 만들어주세요.)
```java
package com.example.myfirstapp.config;

import com.example.myfirstapp.security.JwtAuthorizationFilter;
import com.example.myfirstapp.security.JwtUtil;
import com.example.myfirstapp.security.UserDetailsService;
import lombok.RequiredArgsConstructor;
import org.springframework.boot.autoconfigure.security.servlet.PathRequest;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public JwtAuthorizationFilter jwtAuthorizationFilter() {
        return new JwtAuthorizationFilter(jwtUtil, userDetailsService);
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // CSRF 설정
        http.csrf((csrf) -> csrf.disable());

        // 기본 설정인 Session 방식은 사용하지 않고 JWT 방식을 사용하기 위한 설정
        http.sessionManagement((sessionManagement) ->
                sessionManagement.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        );

        http.authorizeHttpRequests((authorizeHttpRequests) ->
                authorizeHttpRequests
                        .requestMatchers(PathRequest.toStaticResources().atCommonLocations()).permitAll() // resources 접근 허용 설정
                        .requestMatchers("/api/users/**").permitAll() // '/api/user/'로 시작하는 요청 모두 접근 허가
                        .requestMatchers("/h2-console/**").permitAll() // h2-console 접근 허용
                        .anyRequest().authenticated() // 그 외 모든 요청 인증처리
        );
        
        // H2 콘솔 프레임 허용
        http.headers(headers -> headers.frameOptions(frameOptions -> frameOptions.sameOrigin()));

        // 필터 관리
        http.addFilterBefore(jwtAuthorizationFilter(), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

#### **5단계: User 관련 로직 추가 (DTO, Service, Controller)**

**경로:** `src/main/java/com/example/myfirstapp/user/UserRequestDto.java`
```java
package com.example.myfirstapp.user;

import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class UserRequestDto {
    @NotBlank
    private String username;
    @NotBlank
    private String password;
}
```

**경로:** `src/main/java/com/example/myfirstapp/user/UserService.java`
```java
package com.example.myfirstapp.user;

import com.example.myfirstapp.security.JwtUtil;
import lombok.RequiredArgsConstructor;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtUtil jwtUtil;

    public void signup(UserRequestDto requestDto) {
        String username = requestDto.getUsername();
        String password = passwordEncoder.encode(requestDto.getPassword());

        if (userRepository.findByUsername(username).isPresent()) {
            throw new IllegalArgumentException("이미 존재하는 회원입니다.");
        }

        User user = new User(username, password);
        userRepository.save(user);
    }

    public String login(UserRequestDto requestDto) {
        User user = userRepository.findByUsername(requestDto.getUsername())
                .orElseThrow(() -> new IllegalArgumentException("등록된 사용자가 없습니다."));

        if (!passwordEncoder.matches(requestDto.getPassword(), user.getPassword())) {
            throw new IllegalArgumentException("비밀번호가 일치하지 않습니다.");
        }

        return jwtUtil.createToken(user.getUsername());
    }
}
```

**경로:** `src/main/java/com/example/myfirstapp/user/UserController.java`
```java
package com.example.myfirstapp.user;

import jakarta.servlet.http.HttpServletResponse;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @PostMapping("/signup")
    public ResponseEntity<String> signup(@Valid @RequestBody UserRequestDto requestDto) {
        userService.signup(requestDto);
        return ResponseEntity.status(HttpStatus.CREATED).body("회원가입 성공");
    }

    @PostMapping("/login")
    public ResponseEntity<String> login(@RequestBody UserRequestDto requestDto, HttpServletResponse response) {
        String token = userService.login(requestDto);
        response.setHeader(JwtUtil.AUTHORIZATION_HEADER, token);
        return ResponseEntity.ok("로그인 성공");
    }
}
```

#### **6단계: Todo 로직 수정**

이제 `Todo`를 생성, 조회, 수정, 삭제할 때 현재 로그인한 사용자를 기준으로 동작하도록 수정합니다.

**경로:** `src/main/java/com/example/myfirstapp/todo/TodoService.java`
```java
package com.example.myfirstapp.todo;

// ... 기존 import ...
import com.example.myfirstapp.user.User;
import jakarta.persistence.EntityNotFoundException;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class TodoService {

    private final TodoRepository todoRepository;

    @Transactional
    public TodoResponseDto createTodo(TodoRequestDto requestDto, User user) {
        Todo newTodo = new Todo();
        newTodo.setTitle(requestDto.getTitle());
        newTodo.setCompleted(false);
        newTodo.setUser(user); // 사용자 정보 설정

        Todo savedTodo = todoRepository.save(newTodo);
        return new TodoResponseDto(savedTodo);
    }

    public TodoResponseDto getTodoById(Long id, User user) {
        Todo todo = findTodo(id);
        validateTodoOwner(todo, user);
        return new TodoResponseDto(todo);
    }

    public List<TodoResponseDto> getAllTodos(User user) {
        return todoRepository.findAllByUser(user).stream() // 해당 유저의 할 일만 조회
                .map(TodoResponseDto::new)
                .collect(Collectors.toList());
    }

    @Transactional
    public TodoResponseDto updateTodo(Long id, TodoRequestDto requestDto, User user) {
        Todo todo = findTodo(id);
        validateTodoOwner(todo, user);

        if (requestDto.getTitle() != null) {
            todo.setTitle(requestDto.getTitle());
        }
        if (requestDto.getCompleted() != null) {
            todo.setCompleted(requestDto.getCompleted());
        }

        return new TodoResponseDto(todo);
    }

    @Transactional
    public void deleteTodo(Long id, User user) {
        Todo todo = findTodo(id);
        validateTodoOwner(todo, user);
        todoRepository.delete(todo);
    }

    private Todo findTodo(Long id) {
        return todoRepository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("Todo not found with id: " + id));
    }

    private void validateTodoOwner(Todo todo, User user) {
        if (!todo.getUser().getId().equals(user.getId())) {
            throw new SecurityException("You do not have permission to access this todo.");
        }
    }
}
```

**경로:** `src/main/java/com/example/myfirstapp/todo/TodoRepository.java`
`findAllByUser` 메소드를 추가합니다.
```java
package com.example.myfirstapp.todo;

import com.example.myfirstapp.user.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface TodoRepository extends JpaRepository<Todo, Long> {
    List<Todo> findAllByUser(User user); // 메소드 추가
}
```

**경로:** `src/main/java/com/example/myfirstapp/todo/TodoController.java`
컨트롤러의 각 메소드가 인증된 사용자 정보를 `TodoService`로 넘겨주도록 수정합니다.
```java
package com.example.myfirstapp.todo;

import com.example.myfirstapp.security.UserDetailsImpl;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {

    private final TodoService todoService;

    @PostMapping
    public ResponseEntity<TodoResponseDto> createTodo(@RequestBody TodoRequestDto requestDto, @AuthenticationPrincipal UserDetailsImpl userDetails) {
        TodoResponseDto responseDto = todoService.createTodo(requestDto, userDetails.getUser());
        return ResponseEntity.status(HttpStatus.CREATED).body(responseDto);
    }

    @GetMapping("/{id}")
    public ResponseEntity<TodoResponseDto> getTodoById(@PathVariable Long id, @AuthenticationPrincipal UserDetailsImpl userDetails) {
        TodoResponseDto responseDto = todoService.getTodoById(id, userDetails.getUser());
        return ResponseEntity.ok(responseDto);
    }

    @GetMapping
    public ResponseEntity<List<TodoResponseDto>> getAllTodos(@AuthenticationPrincipal UserDetailsImpl userDetails) {
        List<TodoResponseDto> responseDtos = todoService.getAllTodos(userDetails.getUser());
        return ResponseEntity.ok(responseDtos);
    }

    @PutMapping("/{id}")
    public ResponseEntity<TodoResponseDto> updateTodo(@PathVariable Long id, @RequestBody TodoRequestDto requestDto, @AuthenticationPrincipal UserDetailsImpl userDetails) {
        TodoResponseDto responseDto = todoService.updateTodo(id, requestDto, userDetails.getUser());
        return ResponseEntity.ok(responseDto);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTodo(@PathVariable Long id, @AuthenticationPrincipal UserDetailsImpl userDetails) {
        todoService.deleteTodo(id, userDetails.getUser());
        return ResponseEntity.noContent().build();
    }
}
```

#### **7단계: 실행 및 테스트**

이제 모든 준비가 끝났습니다. 애플리케이션을 실행하고 Postman 같은 도구로 테스트합니다.

1.  **회원가입 (POST `/api/users/signup`)**
    ```json
    // Request Body
    {
        "username": "user1",
        "password": "password"
    }
    ```

2.  **로그인 (POST `/api/users/login`)**
    ```json
    // Request Body
    {
        "username": "user1",
        "password": "password"
    }
    ```
    -   **응답 확인:** `Headers` 탭에서 `Authorization` 헤더에 `Bearer ...` 로 시작하는 JWT가 있는지 확인하고 복사합니다.

3.  **할 일 생성 (POST `/api/todos`)**
    -   `Headers` 탭에 `Authorization` 키를 추가하고, 값으로 위에서 복사한 `Bearer ...` 토큰을 붙여넣습니다.
    ```json
    // Request Body
    {
        "title": "내 첫 할일"
    }
    ```

4.  **내 할 일 목록 조회 (GET `/api/todos`)**
    -   `Authorization` 헤더에 토큰을 포함하여 요청하면, 내가 만든 할 일 목록만 보여야 합니다.

5.  **인증 없이 접근 시도 (GET `/api/todos`)**
    -   `Authorization` 헤더 없이 요청하면 `403 Forbidden` 또는 유사한 에러가 발생해야 정상입니다.

---

### **요약 및 참고 자료**

-   **요약:**
    -   Spring Security를 이용해 애플리케이션의 전반적인 보안 설정을 구성했습니다.
    -   회원가입 시 비밀번호를 암호화(`BCryptPasswordEncoder`)하여 저장했습니다.
    -   로그인 성공 시 JWT를 발급하고, 클라이언트는 이후 요청마다 이 토큰을 `Authorization` 헤더에 담아 보냅니다.
    -   서버는 `JwtAuthorizationFilter`를 통해 매 요청마다 토큰을 검증하고 사용자를 인증합니다.
    -   `@AuthenticationPrincipal`을 사용해 컨트롤러에서 인증된 사용자 객체를 쉽게 가져왔습니다.
    -   `Todo`와 `User`를 연관시켜, 서비스 로직에서 할 일 데이터의 소유권을 검사하도록 구현했습니다.

-   **도움이 될 만한 자료:**
    -   **Spring Security 공식 문서:** [https://spring.io/projects/spring-security](https://spring.io/projects/spring-security)
    -   **JWT 소개 (jwt.io):** [https://jwt.io/introduction](https://jwt.io/introduction)
    -   **Baeldung - Spring Security with JWT:** [https://www.baeldung.com/spring-security-oauth-jwt](https://www.baeldung.com/spring-security-oauth-jwt)
