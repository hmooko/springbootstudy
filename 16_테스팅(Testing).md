# 5단계: 배포와 심화 학습 (Deployment & Beyond)

## 1. 테스팅 (Testing)

소프트웨어 개발에서 테스트는 선택이 아닌 필수입니다. 테스트 코드를 작성하면 내가 만든 기능이 의도대로 정확히 동작하는지 검증할 수 있고, 미래에 코드를 수정하거나 새로운 기능을 추가할 때 기존 기능이 망가지는 것을 방지(회귀 테스트)할 수 있습니다. Spring Boot는 테스트를 위한 강력한 기능들을 지원합니다.

테스트는 크게 **단위 테스트(Unit Test)**와 **통합 테스트(Integration Test)**로 나뉩니다.

---

### **1) 단위 테스트 (Unit Test)**

**단위 테스트**는 애플리케이션을 구성하는 가장 작은 단위(메서드나 클래스)가 올바르게 동작하는지 개별적으로 검증하는 것입니다. 중요한 점은 **테스트 대상을 주변 환경으로부터 완벽히 고립**시킨다는 것입니다. 예를 들어, Service 클래스를 테스트한다면, 이와 연결된 Repository는 실제 객체가 아닌 **가짜 객체(Mock)**로 대체하여 테스트를 진행합니다.

-   **장점:** 다른 의존성에 영향을 받지 않으므로 테스트가 단순하고, Spring 컨텍스트를 로드하지 않아 실행 속도가 매우 빠릅니다.
-   **주요 도구:**
    -   **JUnit 5:** Java 진영의 표준 테스트 프레임워크입니다. `@Test`, `@DisplayName` 등의 어노테이션을 사용합니다.
    -   **Mockito:** 가짜 객체(Mock)를 만들어주는 라이브러리입니다. `@Mock`, `@InjectMocks` 등의 어노테이션을 사용합니다.

#### **단위 테스트 코드 예시 (Service 테스트)**

`TodoService`가 `TodoRepository`를 사용하여 할 일을 저장하는 로직을 테스트하는 상황을 가정해봅시다.

```java
// TodoService.java (테스트 대상)
@Service
public class TodoService {
    private final TodoRepository todoRepository;

    public TodoService(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    public Todo createTodo(String content) {
        if (content == null || content.isBlank()) {
            throw new IllegalArgumentException("내용을 입력해주세요.");
        }
        Todo newTodo = new Todo(content);
        return todoRepository.save(newTodo);
    }
}

// TodoServiceTest.java (테스트 코드)
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.mockito.Mockito.*;
import static org.assertj.core.api.Assertions.*;

@ExtendWith(MockitoExtension.class) // JUnit5에서 Mockito를 사용하기 위한 설정
class TodoServiceTest {

    @Mock // 가짜 TodoRepository 객체 생성
    private TodoRepository todoRepository;

    @InjectMocks // @Mock으로 만든 객체를 TodoService에 주입
    private TodoService todoService;

    @Test
    void createTodo_성공() {
        // given (준비)
        String content = "새로운 할 일";
        Todo todoToSave = new Todo(content);
        // todoRepository.save()가 호출되면, 저장된 것처럼 객체를 반환하도록 설정
        when(todoRepository.save(any(Todo.class))).thenReturn(todoToSave);

        // when (실행)
        Todo createdTodo = todoService.createTodo(content);

        // then (검증)
        assertThat(createdTodo.getContent()).isEqualTo(content);
        // todoRepository의 save 메서드가 정확히 1번 호출되었는지 검증
        verify(todoRepository, times(1)).save(any(Todo.class));
    }
}
```

---

### **2) 통합 테스트 (Integration Test)**

**통합 테스트**는 여러 컴포넌트(Controller, Service, Repository 등)를 함께 묶어, 전체 시스템이 의도대로 동작하는지 검증하는 것입니다. 실제 Spring 컨테이너를 실행하여 모든 Bean을 등록하고 테스트하므로, 단위 테스트보다 훨씬 넓은 범위를 다룹니다.

-   **장점:** 실제 운영 환경과 가장 유사한 조건에서 테스트하므로, 각 컴포넌트의 상호작용 중에 발생하는 문제를 찾아낼 수 있습니다.
-   **단점:** Spring 컨텍스트를 모두 로드하고 실제 DB에 접근하기도 하므로 실행 속도가 느립니다.
-   **주요 도구:**
    -   `@SpringBootTest`: 통합 테스트를 위해 전체 Spring Boot 설정을 로드합니다.
    -   `@AutoConfigureMockMvc`: `MockMvc`를 사용하여 실제 HTTP 요청을 보내는 것처럼 컨트롤러를 테스트할 수 있게 해줍니다.
    -   `@Transactional`: 테스트가 끝난 후 데이터베이스의 변경사항을 모두 롤백하여, 각 테스트가 서로 영향을 주지 않도록 합니다.

#### **통합 테스트 코드 예시 (Controller 테스트)**

`TodoController`의 "할 일 생성" API가 올바르게 동작하는지 테스트하는 상황입니다.

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.transaction.annotation.Transactional;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest // 통합 테스트 설정
@AutoConfigureMockMvc // MockMvc 주입을 위한 설정
@Transactional // 테스트 후 DB 롤백
class TodoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void POST_api_todos_성공() throws Exception {
        // given
        String requestJson = "{\"content\":\"새로운 할 일\"}";

        // when & then
        mockMvc.perform(post("/api/todos") // POST /api/todos 요청을 보냄
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isCreated()) // HTTP 상태 코드가 201 Created인지 확인
            .andExpect(jsonPath("$.content").value("새로운 할 일")); // 응답 JSON의 content 필드 값 확인
    }
}
```

---

### **요약: 단위 테스트 vs 통합 테스트**

| 구분 | 단위 테스트 | 통합 테스트 |
| :--- | :--- | :--- |
| **테스트 대상** | 단일 컴포넌트 (클래스, 메서드) | 여러 컴포넌트의 상호작용 |
| **의존성 처리** | 가짜 객체(Mock)로 대체 | 실제 객체(Bean)를 주입 |
| **Spring 컨텍스트** | 로드하지 않음 | 전체를 로드함 |
| **실행 속도** | 매우 빠름 | 느림 |
| **목적** | 기능의 논리적 정확성 검증 | 시스템의 전체 흐름 및 연동 검증 |

**테스트 피라미드 전략**에 따라, 빠르고 작성하기 쉬운 **단위 테스트를 많이 작성**하고, 상대적으로 느리고 무거운 **통합 테스트는 핵심적인 API 흐름 위주로 작성**하는 것이 효율적입니다.

---

### **출처 및 참고 자료**

-   **공식 문서:**
    -   Spring Boot Testing: [https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)
-   **개념 이해를 위한 블로그:**
    -   Testing in Spring Boot (Baeldung): [https://www.baeldung.com/spring-boot-testing](https://www.baeldung.com/spring-boot-testing)
    -   Guide to @SpringBootTest (Reflectoring): [https://reflectoring.io/spring-boot-test/](https://reflectoring.io/spring-boot-test/)
    -   Spring Boot, Mockito and JUnit 5 Example (HowToDoInJava): [https://howtodoinjava.com/spring-boot2/testing/spring-boot-mockito-junit5-example/](https://howtodoinjava.com/spring-boot2/testing/spring-boot-mockito-junit5-example/)

