# 4단계: 실전 기능 추가 (Advanced Features)

## 3. 유효성 검사 (Validation)

클라이언트로부터 들어오는 데이터가 항상 우리가 원하는 형식과 값을 가지고 있다고 보장할 수 없습니다. **유효성 검사(Validation)**는 애플리케이션을 잘못된 데이터로부터 보호하고, 데이터의 정합성을 보장하기 위한 필수적인 과정입니다. Spring Boot는 `spring-boot-starter-validation`을 통해 Java의 표준 기술인 **Bean Validation**을 매우 쉽게 사용할 수 있도록 지원합니다.

---

### **왜 유효성 검사가 중요한가?**

-   **데이터 무결성 보호:** 데이터베이스나 시스템에 의도하지 않은, 잘못된 데이터가 저장되는 것을 막습니다.
-   **보안 강화:** 예상치 못한 형식의 입력값으로 인해 발생할 수 있는 보안 취약점(예: SQL Injection)을 예방하는 데 도움이 됩니다.
-   **명확한 피드백:** 사용자에게 어떤 값이 잘못되었는지 명확하게 알려주어 사용자 경험(UX)을 향상시킬 수 있습니다.

---

### **사용 방법**

#### **1단계: 의존성 확인**

`build.gradle` 또는 `pom.xml`에 `spring-boot-starter-web` 의존성이 포함되어 있다면, `spring-boot-starter-validation`이 기본적으로 포함되어 있으므로 별도의 설정이 필요 없습니다.

```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // spring-boot-starter-web 안에 validation 스타터가 포함되어 있음
}
```

#### **2단계: DTO에 유효성 검사 어노테이션 추가**

클라이언트의 요청 데이터를 받을 DTO 클래스의 각 필드에 어떤 값이 유효한지를 어노테이션으로 선언합니다.

```java
// 예: 회원가입 요청 DTO
public record SignupRequestDto(
    @NotBlank(message = "사용자 이름은 비워둘 수 없습니다.")
    String username,

    @NotBlank(message = "비밀번호는 비워둘 수 없습니다.")
    @Size(min = 8, max = 20, message = "비밀번호는 8자 이상 20자 이하로 입력해주세요.")
    String password,

    @NotBlank(message = "이메일은 비워둘 수 없습니다.")
    @Email(message = "유효한 이메일 형식이 아닙니다.")
    String email
) {}
```

#### **3단계: 컨트롤러에서 `@Valid` 어노테이션 사용**

컨트롤러의 메서드에서 요청 데이터를 받는 파라미터(`@RequestBody`가 붙은 DTO) 앞에 `@Valid` 어노테이션을 붙여줍니다. 이렇게 하면 해당 DTO에 설정된 제약 조건에 따라 유효성 검사가 자동으로 수행됩니다.

```java
@RestController
public class AuthController {

    @PostMapping("/api/auth/signup")
    public ResponseEntity<String> signup(@Valid @RequestBody SignupRequestDto signupRequestDto) {
        // 유효성 검사를 통과하면 이 코드가 실행됨
        // ... 회원가입 비즈니스 로직 ...
        return ResponseEntity.ok("회원가입이 완료되었습니다.");
    }
}
```

---

### **유효성 검사 실패 시 처리 방법 (예외 처리와의 연계)**

만약 유효성 검사에 실패하면, Spring Boot는 **`MethodArgumentNotValidException`** 이라는 예외를 발생시킵니다. 이 예외를 우리가 이전에 만들었던 `GlobalExceptionHandler`에서 처리하도록 코드를 추가하면, 모든 유효성 검사 오류를 한 곳에서 관리하고 일관된 형식으로 응답할 수 있습니다.

#### **`GlobalExceptionHandler`에 핸들러 메서드 추가**

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // ... (기존에 있던 다른 ExceptionHandler 메서드들) ...

    // @Valid 유효성 검사 실패 시 발생하는 예외 처리
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleMethodArgumentNotValid(MethodArgumentNotValidException ex) {
        // 실패한 유효성 검사에 대한 첫 번째 오류 메시지를 가져옵니다.
        String errorMessage = ex.getBindingResult().getAllErrors().get(0).getDefaultMessage();
        
        ErrorResponse response = new ErrorResponse("INVALID_INPUT_VALUE", errorMessage);
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST); // 400 Bad Request
    }
}
```

이제 클라이언트가 비밀번호를 7자리만 입력하는 등 유효성 검사에 실패하는 요청을 보내면, `GlobalExceptionHandler`가 `MethodArgumentNotValidException`을 잡아내어 `{"errorCode":"INVALID_INPUT_VALUE", "message":"비밀번호는 8자 이상 20자 이하로 입력해주세요."}` 와 같은 깔끔한 JSON 응답을 자동으로 보내주게 됩니다.

---

### **자주 사용되는 유효성 검사 어노테이션**

| 어노테이션 | 설명 |
| :--- | :--- |
| `@NotNull` | `null` 값만 허용하지 않음 (`""` 이나 `" "`는 허용) |
| `@NotEmpty` | `null`과 빈 문자열(`""`)을 허용하지 않음 (`" "`는 허용) |
| `@NotBlank` | `null`, 빈 문자열, 공백만 있는 문자열 모두 허용하지 않음 (가장 강력) |
| `@Size(min, max)` | 문자열, 배열, 컬렉션의 크기가 지정된 범위 안에 있어야 함 |
| `@Min(value)` | 숫자 값이 지정된 값 이상이어야 함 |
| `@Max(value)` | 숫자 값이 지정된 값 이하여야 함 |
| `@Email` | 유효한 이메일 주소 형식이어야 함 |
| `@Pattern(regexp)` | 값이 지정된 정규 표현식과 일치해야 함 |

---

### **요약**

-   **유효성 검사**는 DTO의 필드에 `@NotBlank`, `@Size` 등의 **어노테이션을 추가**하여 정의합니다.
-   컨트롤러 메서드에서 `@RequestBody`로 받는 DTO 앞에 **`@Valid` 어노테이션**을 붙여 검사를 실행합니다.
-   유효성 검사 실패 시 **`MethodArgumentNotValidException`** 이 발생합니다.
-   이 예외를 **`@RestControllerAdvice`** 를 이용한 전역 예외 처리기에서 핸들링하여, 사용자에게 일관된 오류 메시지를 응답하는 것이 가장 좋은 방법입니다.

---

### **출처 및 참고 자료**

-   **공식 문서:**
    -   Jakarta Bean Validation-Annotations: [https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/package-summary](https://jakarta.ee/specifications/bean-validation/3.0/apidocs/jakarta/validation/constraints/package-summary)
-   **개념 이해를 위한 블로그:**
    -   Validation in Spring Boot (Baeldung): [https://www.baeldung.com/spring-boot-bean-validation](https://www.baeldung.com/spring-boot-bean-validation)
    -   [Spring] @Valid를 이용한 유효성 검증 (velog): [https://velog.io/@codemcd/Spring-Valid를-이용한-유효성-검증](https://velog.io/@codemcd/Spring-Valid%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%9C%A0%ED%9A%A8%EC%84%B1-%EA%B2%80%EC%A6%9D)
