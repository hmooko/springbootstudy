### **미니 프로젝트 #3: 간단한 To-Do List API 만들기**

#### **1단계: 의존성 설정 (`build.gradle`)**

먼저 프로젝트에 필요한 라이브러리들이 있는지 확인하고 추가해야 합니다. `build.gradle` 파일의 `dependencies` 블록에 아래 세 가지가 포함되어 있는지 확인해주세요. 없다면 추가합니다.

-   `spring-boot-starter-web`: 웹 애플리케이션과 API를 만들기 위한 필수 라이브러리
-   `spring-boot-starter-data-jpa`: Spring Data JPA와 Hibernate를 사용하기 위한 라이브러리
-   `h2database`: H2 인메모리 데이터베이스 라이브러리
-   `lombok`: 어노테이션으로 반복 코드를 줄여주는 라이브러리 (선택사항이지만 강력 추천)

```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    // ... 기타 의존성
}
```

#### **2단계: H2 데이터베이스 설정 (`application.properties`)**

다음으로, H2 데이터베이스를 활성화하고 개발 중에 데이터베이스 상태를 쉽게 확인할 수 있는 H2 콘솔을 사용하도록 설정합니다.
`src/main/resources/application.properties` 파일에 아래 내용을 추가하세요.

```properties
# H2 Database Settings
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Datasource Settings
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA Settings
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```
- `spring.h2.console.enabled=true`: H2 콘솔을 활성화합니다.
- `spring.datasource.url=jdbc:h2:mem:testdb`: 데이터베이스를 메모리 모드(애플리케이션 실행 시 생성, 종료 시 삭제)로 실행합니다.
- `spring.jpa.hibernate.ddl-auto=update`: 애플리케이션 실행 시 Entity 클래스를 보고 데이터베이스 테이블을 자동으로 생성/수정합니다.
- `spring.jpa.show-sql=true`: JPA가 실행하는 SQL 쿼리를 로그로 보여줍니다.

#### **3단계: `Todo` 엔티티 만들기**

데이터베이스의 `todo` 테이블과 매핑될 `Todo` 클래스를 만듭니다.

**경로:** `src/main/java/com/example/myfirstapp/todo/Todo.java`
(만약 `todo` 패키지가 없다면 새로 만들어주세요.)

```java
package com.example.myfirstapp.todo;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Todo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    private boolean completed;
}
```

#### **4단계: `TodoRepository` 만들기**

`Todo` 엔티티의 데이터베이스 작업을 처리할 리포지토리 인터페이스를 만듭니다.

**경로:** `src/main/java/com/example/myfirstapp/todo/TodoRepository.java`

```java
package com.example.myfirstapp.todo;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface TodoRepository extends JpaRepository<Todo, Long> {
}
```
이것만으로 기본적인 CRUD 기능은 모두 완성되었습니다.

#### **5단계: DTO 만들기**

API의 요청(Request)과 응답(Response)에 사용할 객체를 만듭니다.

**경로:** `src/main/java/com/example/myfirstapp/todo/TodoRequestDto.java`
```java
package com.example.myfirstapp.todo;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
public class TodoRequestDto {
    private String title;
    private Boolean completed;
}
```

**경로:** `src/main/java/com/example/myfirstapp/todo/TodoResponseDto.java`
```java
package com.example.myfirstapp.todo;

import lombok.Getter;

@Getter
public class TodoResponseDto {
    private final Long id;
    private final String title;
    private final boolean completed;

    public TodoResponseDto(Todo todo) {
        this.id = todo.getId();
        this.title = todo.getTitle();
        this.completed = todo.isCompleted();
    }
}
```

#### **6단계: `TodoService` 만들기**

비즈니스 로직을 처리할 서비스 클래스를 만듭니다.

**경로:** `src/main/java/com/example/myfirstapp/todo/TodoService.java`
```java
package com.example.myfirstapp.todo;

import jakarta.persistence.EntityNotFoundException;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor // final 필드에 대한 생성자를 자동으로 만들어줍니다.
@Transactional(readOnly = true) // 기본적으로 모든 메소드에 읽기 전용 트랜잭션 적용
public class TodoService {

    private final TodoRepository todoRepository;

    @Transactional // 이 메소드는 데이터를 변경하므로 readOnly=false(기본값) 적용
    public TodoResponseDto createTodo(TodoRequestDto requestDto) {
        Todo newTodo = new Todo();
        newTodo.setTitle(requestDto.getTitle());
        newTodo.setCompleted(false); // 새로 생성된 할 일은 항상 미완료 상태

        Todo savedTodo = todoRepository.save(newTodo);
        return new TodoResponseDto(savedTodo);
    }

    public TodoResponseDto getTodoById(Long id) {
        Todo todo = todoRepository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("Todo not found with id: " + id));
        return new TodoResponseDto(todo);
    }

    public List<TodoResponseDto> getAllTodos() {
        return todoRepository.findAll().stream()
                .map(TodoResponseDto::new)
                .collect(Collectors.toList());
    }

    @Transactional
    public TodoResponseDto updateTodo(Long id, TodoRequestDto requestDto) {
        Todo todo = todoRepository.findById(id)
                .orElseThrow(() -> new EntityNotFoundException("Todo not found with id: " + id));

        // 제목이나 완료 여부가 요청에 포함된 경우에만 업데이트
        if (requestDto.getTitle() != null) {
            todo.setTitle(requestDto.getTitle());
        }
        if (requestDto.getCompleted() != null) {
            todo.setCompleted(requestDto.getCompleted());
        }

        return new TodoResponseDto(todo); // 변경 감지(Dirty Checking)에 의해 자동 업데이트됨
    }

    @Transactional
    public void deleteTodo(Long id) {
        todoRepository.deleteById(id);
    }
}
```

#### **7단계: `TodoController` 만들기**

API 엔드포인트를 정의하는 컨트롤러를 만듭니다.

**경로:** `src/main/java/com/example/myfirstapp/todo/TodoController.java`
```java
package com.example.myfirstapp.todo;

import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/todos")
@RequiredArgsConstructor
public class TodoController {

    private final TodoService todoService;

    @PostMapping
    public ResponseEntity<TodoResponseDto> createTodo(@RequestBody TodoRequestDto requestDto) {
        TodoResponseDto responseDto = todoService.createTodo(requestDto);
        return ResponseEntity.status(HttpStatus.CREATED).body(responseDto);
    }

    @GetMapping("/{id}")
    public ResponseEntity<TodoResponseDto> getTodoById(@PathVariable Long id) {
        TodoResponseDto responseDto = todoService.getTodoById(id);
        return ResponseEntity.ok(responseDto);
    }

    @GetMapping
    public ResponseEntity<List<TodoResponseDto>> getAllTodos() {
        List<TodoResponseDto> responseDtos = todoService.getAllTodos();
        return ResponseEntity.ok(responseDtos);
    }

    @PutMapping("/{id}")
    public ResponseEntity<TodoResponseDto> updateTodo(@PathVariable Long id, @RequestBody TodoRequestDto requestDto) {
        TodoResponseDto responseDto = todoService.updateTodo(id, requestDto);
        return ResponseEntity.ok(responseDto);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTodo(@PathVariable Long id) {
        todoService.deleteTodo(id);
        return ResponseEntity.noContent().build();
    }
}
```

#### **8단계: 실행 및 테스트**

이제 애플리케이션을 실행하고 API가 잘 동작하는지 확인합니다. `curl`이나 Postman 같은 도구를 사용할 수 있습니다.

-   **애플리케이션 실행:** `...Application.java` 파일의 `main` 메소드를 실행합니다.
-   **H2 콘솔 확인 (선택사항):** 웹 브라우저에서 `http://localhost:8080/h2-console` 로 접속하여 DB 상태를 볼 수 있습니다. (JDBC URL은 `jdbc:h2:mem:testdb` 로 입력)

**`curl` 테스트 명령어 예시:**

1.  **할 일 생성 (POST):**
    ```bash
    curl -X POST http://localhost:8080/api/todos -H "Content-Type: application/json" -d '{"title": "Spring Boot 공부하기"}'
    ```

2.  **전체 할 일 조회 (GET):**
    ```bash
    curl http://localhost:8080/api/todos
    ```

3.  **특정 할 일 조회 (GET):** (id가 1인 경우)
    ```bash
    curl http://localhost:8080/api/todos/1
    ```

4.  **할 일 수정 (PUT):** (id가 1인 할 일을 완료 처리)
    ```bash
    curl -X PUT http://localhost:8080/api/todos/1 -H "Content-Type: application/json" -d '{"completed": true}'
    ```

5.  **할 일 삭제 (DELETE):** (id가 1인 할 일 삭제)
    ```bash
    curl -X DELETE http://localhost:8080/api/todos/1
    ```
