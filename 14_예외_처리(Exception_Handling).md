# 4단계: 실전 기능 추가 (Advanced Features)

## 2. 예외 처리 (Exception Handling)

잘 만든 애플리케이션은 기능이 정상적으로 동작하는 것뿐만 아니라, 오류가 발생했을 때 사용자에게 명확하고 일관된 피드백을 주는 것도 매우 중요합니다. Spring Boot에서는 `@ControllerAdvice`와 `@ExceptionHandler`를 사용하여 애플리케이션 전역의 예외를 깔끔하게 처리할 수 있습니다.

---

### **왜 전역 예외 처리가 필요한가?**

만약 모든 서비스 메서드나 컨트롤러 메서드에서 `try-catch` 블록으로 예외를 처리한다면, 다음과 같은 문제가 발생합니다.

-   **코드 중복:** 비슷한 형태의 `try-catch` 코드가 여러 곳에 흩어져 유지보수가 어렵습니다.
-   **처리 누락:** 개발자가 실수로 특정 예외에 대한 처리를 누락할 수 있습니다.
-   **일관성 없는 응답:** 예외마다 다른 형식의 오류 메시지를 반환하게 되어 클라이언트가 대응하기 어렵습니다.

**전역 예외 처리(Global Exception Handling)**는 이러한 문제들을 해결하기 위해, 예외 처리 로직을 한 곳으로 모아 중앙에서 관리하는 방식입니다.

---

### **핵심 어노테이션: `@RestControllerAdvice` 와 `@ExceptionHandler`**

-   **`@RestControllerAdvice`**: `@ControllerAdvice`와 `@ResponseBody`를 합친 어노테이션입니다. 이 어노테이션이 붙은 클래스는 애플리케이션의 모든 `@RestController`에서 발생하는 예외를 감지하고 처리하는 역할을 합니다. 반환값은 JSON 형태로 만들어집니다.

-   **`@ExceptionHandler({처리할_예외.class})`**: 특정 예외가 발생했을 때 실행될 메서드를 지정하는 어노테이션입니다. `@RestControllerAdvice` 클래스 내부에 이 어노테이션을 붙인 메서드를 만들어 예외 종류별로 다른 처리를 하도록 구현합니다.

---

### **실전 코드 예시**

#### **1단계: 일관된 오류 응답을 위한 `ErrorResponse` DTO 만들기**

클라이언트에게 항상 동일한 형식의 JSON으로 오류를 알려주기 위해, 오류 응답을 위한 DTO(Data Transfer Object)를 만듭니다.

```java
// record (Java 16+) 또는 일반 class로 생성 가능
public record ErrorResponse(
    String errorCode, // 직접 정의하는 에러 코드
    String message    // 사용자에게 보여줄 오류 메시지
) {}
```

#### **2단계: 상황에 맞는 `CustomException` 만들기**

"해당 사용자를 찾을 수 없습니다" 와 같이, 비즈니스 로직 상에서 발생하는 특정 상황을 표현하는 예외 클래스를 직접 만들면 좋습니다. 이렇게 하면 예외의 의도가 명확해지고, 종류별로 다른 처리가 가능해집니다.

```java
// RuntimeException을 상속받는 것이 일반적입니다.
public class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(String message) {
        super(message);
    }
}
```

#### **3단계: 모든 예외를 처리하는 `GlobalExceptionHandler` 만들기**

이제 `@RestControllerAdvice`를 사용하여 전역 예외 처리기를 만듭니다.

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // 직접 만든 UserNotFoundException 예외 처리
    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFoundException(UserNotFoundException ex) {
        ErrorResponse response = new ErrorResponse("USER_NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(response, HttpStatus.NOT_FOUND); // 404 Not Found
    }

    // 일반적인 비즈니스 로직 예외 처리
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(IllegalArgumentException ex) {
        ErrorResponse response = new ErrorResponse("INVALID_ARGUMENT", ex.getMessage());
        return new ResponseEntity<>(response, HttpStatus.BAD_REQUEST); // 400 Bad Request
    }

    // 위에서 처리하지 못한 모든 예외를 처리 (최후의 보루)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAllUncaughtException(Exception ex) {
        // 실제 운영에서는 로그를 더 상세히 기록해야 합니다.
        System.err.println("Unhandled Exception: " + ex.getMessage());
        ex.printStackTrace();

        ErrorResponse response = new ErrorResponse("INTERNAL_SERVER_ERROR", "서버 내부 오류가 발생했습니다.");
        return new ResponseEntity<>(response, HttpStatus.INTERNAL_SERVER_ERROR); // 500 Internal Server Error
    }
}
```

이제 서비스나 컨트롤러에서는 `throw new UserNotFoundException("ID가 123인 사용자를 찾을 수 없습니다.");` 와 같이 예외를 던지기만 하면, `GlobalExceptionHandler`가 알아서 적절한 HTTP 상태 코드와 JSON 응답을 생성해줍니다.

---

### **요약**

-   **전역 예외 처리**는 코드 중복을 줄이고, 일관된 오류 응답을 제공하여 애플리케이션의 안정성과 유지보수성을 높입니다.
-   `@RestControllerAdvice` 어노테이션을 클래스에 붙여 **전역 예외 처리기**를 만듭니다.
-   `@ExceptionHandler` 어노테이션을 메서드에 붙여 **특정 예외를 잡아서 처리**하는 로직을 구현합니다.
-   `ErrorResponse` DTO를 만들어 **일관된 JSON 오류 메시지 형식**을 클라이언트에게 반환하는 것이 좋습니다.

---

### **출처 및 참고 자료**

-   **공식 문서:**
    -   Error Handling in Spring Boot: [https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc](https://spring.io/blog/2013/11/01/exception-handling-in-spring-mvc)
-   **개념 이해를 위한 블로그:**
    -   @ControllerAdvice/@RestControllerAdvice로 예외처리하기 (우아한형제들 기술블로그): [https://techblog.woowahan.com/2597/](https://techblog.woowahan.com/2597/)
    -   스프링의 다양한 예외 처리 방법 완벽하게 이해하기 (망나니개발자): [https://mangkyu.tistory.com/204](https://mangkyu.tistory.com/204)
    -   스프링 부트 전역 예외 처리 최적화 하기 (Velog): [https://velog.io/@auth/Spring-Boot-Exception-Handling](https://velog.io/@auth/Spring-Boot-Exception-Handling)
