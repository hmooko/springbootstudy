### **2-4. 기본 API 만들기**

가장 기본적인 "Hello, World!" 메시지를 반환하는 API를 만들면서, Spring Boot가 어떻게 웹 요청을 처리하는지 그 원리를 이해해 보겠습니다.

#### **1. 컨트롤러(Controller) 생성하기**

Spring Boot에서 **컨트롤러**는 웹 요청의 '교통정리' 역할을 합니다. 특정 URL로 요청이 들어오면, 그 요청을 받아서 처리할 메소드를 연결해주는 클래스입니다.

개발의 편의와 코드의 구조화를 위해, 컨트롤러는 별도의 패키지에 모아두는 것이 좋습니다.

1.  IDE의 프로젝트 탐색기에서 `src/main/java/내 패키지 경로` 로 이동합니다.
2.  해당 패키지 위에서 마우스 오른쪽 클릭 > `New` > `Package`를 선택하여 `controller` 라는 이름의 새 패키지를 만듭니다.
	1. 
3.  방금 만든 `controller` 패키지 안에서, `HelloController` 라는 이름의 새 Java 클래스를 생성합니다.

#### **2. "Hello, World!" 코드 작성하기**

방금 만든 `HelloController.java` 파일을 열고, 아래와 같이 코드를 작성하세요.

```java
package com.example.myfirstapp.controller; // 여러분의 패키지 경로에 맞게 표시됩니다.

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

	// GET https://naver.com/hello
    @GetMapping("/hello")
    public String sayHello() {
        return "Hello, World!";
    }
}
```

#### **3. 코드 해설: 마법 같은 어노테이션(Annotation)**

위 코드에서 핵심은 `@RestController`와 `@GetMapping`이라는 어노테이션입니다.

-   `@RestController`
    이 어노테이션을 클래스에 붙이면, Spring Framework는 "아, 이 클래스는 REST API 요청을 처리하는 컨트롤러구나!"라고 인지합니다. 이 클래스 내부의 메소드들은 HTML 페이지 같은 뷰(View)를 반환하는 대신, 문자열, JSON 객체 등 데이터 자체를 HTTP 응답 본문(Response Body)에 직접 써서 반환하게 됩니다.

-   `@GetMapping("/hello")`
    이 어노테이션은 HTTP의 GET 요청을 특정 URL 주소와 매핑(mapping, 연결)하는 역할을 합니다. 즉, 클라이언트(웹 브라우저 등)가 우리 서버의 `/hello` 라는 주소로 GET 요청을 보내면, Spring이 `@GetMapping("/hello")`가 붙어있는 `sayHello()` 메소드를 찾아서 실행시켜 줍니다.

#### **4. 실행 및 테스트**

1.  이전 단계에서 했던 것처럼, `...Application.java` 파일의 `main` 메소드를 실행하여 Spring Boot 애플리케이션을 시작합니다.
2.  웹 브라우저을 열고 주소창에 `http://localhost:8080/hello` 를 입력하고 Enter 키를 누릅니다.
3.  브라우저 화면에 "Hello, World!" 라는 문구가 나타나면 성공적으로 첫 API를 만든 것입니다!

#### **5. 심화: 다양한 요청 처리 방법**

실제 애플리케이션에서는 클라이언트로부터 데이터를 입력받는 경우가 대부분입니다. Spring Boot는 이를 위한 편리한 어노테이션들을 제공합니다.

-   **`@RequestParam`**: URL의 쿼리 파라미터(Query Parameter) 값을 받을 때 사용합니다.
    -   **요청 URL 예시:** `http://localhost:8080/api/greet?name=Gemini`
    -   **코드 예시:**
        ```java
        @GetMapping("/api/greet")
        public String greetByName(@RequestParam("name") String visitorName) {
            return "Hello, " + visitorName + "!";
        }
        ```

-   **`@PathVariable`**: URL 경로(path) 자체에 포함된 값을 변수로 받을 때 사용합니다.
    -   **요청 URL 예시:** `http://localhost:8080/users/123` (123이라는 사용자 ID를 조회)
    -   **코드 예시:**
        ```java
        @GetMapping("/users/{userId}")
        public String getUserInfo(@PathVariable("userId") Long id) {
            return "Requesting info for user ID: " + id;
        }
        ```

-   **`@RequestBody`**: 요청의 본문(Body)에 담겨 오는 데이터(주로 JSON)를 Java 객체로 자동 변환해줍니다.
    -   **요청 예시:** `POST` 방식으로 `/users` 에 아래 JSON 데이터를 담아 요청
        ```json
        {
            "name": "Thomas",
            "email": "thomas@google.com"
        }
        ```
    -   **코드 예시:** (먼저 `User` 라는 데이터 클래스가 필요합니다)
        ```java
        // 1. 데이터를 담을 User 클래스 (DTO)
        public class User {
            private String name;
            private String email;
            // Getter, Setter...
        }

        // 2. 컨트롤러 메소드
        @PostMapping("/users")
        public String createUser(@RequestBody User user) {
            return "New user created: " + user.getName() + ", Email: " + user.getEmail();
        }
        ```

---

### **요약**

-   `@RestController` 어노테이션으로 API 요청을 처리할 컨트롤러 클래스를 지정합니다.
-   `@GetMapping`, `@PostMapping` 등으로 HTTP 요청 방식과 URL 경로를 처리할 메소드에 연결(매핑)합니다.
-   클라이언트가 보낸 데이터는 `@RequestParam`, `@PathVariable`, `@RequestBody` 등을 사용하여 편리하게 받을 수 있습니다.

### **출처 및 참고 자료**

-   **공식 문서:**
    - Spring Web on IntelliJ IDEA: [https://www.jetbrains.com/idea/guide/tutorials/spring-boot-hello-world/spring-boot-web-app/](https://www.jetbrains.com/idea/guide/tutorials/spring-boot-hello-world/spring-boot-web-app/)
-   **추천 블로그:**
    - GeeksforGeeks - Spring Boot @RestController Annotation: [https://www.geeksforgeeks.org/spring-boot-restcontroller-annotation/](https://www.geeksforgeeks.org/spring-boot-restcontroller-annotation/)
    - Baeldung - Introduction to Spring @RequestBody and @ResponseBody: [https://www.baeldung.com/spring-request-response-body](https://www.baeldung.com/spring-request-response-body)
