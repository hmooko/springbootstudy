### **3-4. 계층형 아키텍처: Controller - Service - Repository**

애플리케이션의 기능이 점점 많아지면, 한 클래스에 모든 코드를 작성하는 것은 유지보수 재앙으로 이어집니다. 코드를 역할과 책임에 따라 여러 계층으로 나누어 구성하는 것이 매우 중요합니다. Spring Boot에서는 일반적으로 다음과 같은 3계층 구조를 사용합니다.

!https://blog.kakaocdn.net/dn/bO3U95/btq9e2y2j0K/kK6k1kE4kK1kK1kK1kK1kK/img.png

#### **1. 왜 계층을 나누어야 하는가?**

-   **관심사의 분리 (Separation of Concerns):** 각 계층은 자신만의 역할에만 집중합니다. 코드가 간결해지고 이해하기 쉬워집니다.
-   **유지보수 용이성:** 특정 기능을 수정할 때, 관련된 계층의 코드만 보면 되므로 수정 범위가 명확해집니다. 예를 들어, 데이터베이스 종류를 바꾸면 Repository 계층만 수정하면 됩니다.
-   **테스트 용이성:** 각 계층을 독립적으로 테스트하기 쉬워집니다. 서비스 로직을 테스트할 때, 실제 DB가 없어도 가짜(Mock) Repository를 만들어 테스트할 수 있습니다.

#### **2. 각 계층의 역할과 책임**

**① Repository (저장소 계층)**

-   **역할:** 데이터베이스와 직접 통신하는 최하위 계층입니다.
-   **책임:** 오직 데이터의 CRUD(생성, 조회, 수정, 삭제) 작업에만 집중합니다.
-   **구현:** Spring Data JPA를 사용하여 `JpaRepository`를 상속한 인터페이스로 만듭니다.
-   **어노테이션:** `@Repository` (Spring Data JPA에서는 생략 가능)

```java
// Repository Layer
public interface MemberRepository extends JpaRepository<Member, Long> {
    // findByEmail() 같은 쿼리 메소드만 정의
}
```

**② Service (서비스 계층)**

-   **역할:** 애플리케이션의 핵심 비즈니스 로직을 처리하는 '두뇌'와 같은 계층입니다.
-   **책임:**
    -   Controller로부터 요청을 받아 비즈니스 규칙에 맞게 데이터를 처리합니다.
    -   필요한 경우 여러 Repository를 호출하여 데이터를 조합하거나, 트랜잭션(Transaction)을 관리합니다.
    -   Controller와 Repository 사이의 중재자 역할을 합니다.
-   **구현:** 비즈니스 로직을 담은 메소드들로 구성된 클래스로 만듭니다.
-   **어노테이션:** `@Service`

```java
// Service Layer
@Service
public class MemberService {
    private final MemberRepository memberRepository;

    // 생성자를 통한 의존성 주입 (Dependency Injection)
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public Member createMember(Member member) {
        // 예: 이메일 중복 확인 같은 비즈니스 로직 수행
        return memberRepository.save(member);
    }
}
```

**③ Controller (컨트롤러 계층)**

-   **역할:** 외부(클라이언트)의 HTTP 요청을 가장 먼저 받는 '관문'입니다.
-   **책임:**
    -   HTTP 요청을 받아 파라미터를 파싱하고 유효성을 검사합니다.
    -   적절한 Service 메소드를 호출하여 작업 처리를 위임합니다.
    -   Service로부터 받은 결과를 클라이언트에게 반환할 HTTP 응답(JSON 등)으로 가공하여 보냅니다.
    -   **절대 비즈니스 로직을 포함해서는 안 됩니다.**
-   **구현:** `@RestController` 어노테이션과 `@GetMapping`, `@PostMapping` 등으로 API 엔드포인트를 정의한 클래스로 만듭니다.
-   **어노테이션:** `@RestController`

```java
// Controller Layer
@RestController
public class MemberController {
    private final MemberService memberService;

    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @PostMapping("/members")
    public ResponseEntity<Member> postMember(@RequestBody Member member) {
        Member createdMember = memberService.createMember(member);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdMember);
    }
}
```

#### **3. DTO (Data Transfer Object)의 중요성**

위 예제에서는 Controller, Service, Repository가 모두 `Member`라는 동일한 객체(JPA Entity)를 사용했습니다. 하지만 이는 좋지 않은 설계입니다. **Entity는 데이터베이스 테이블 구조와 직접 연결된 매우 내밀한 객체이므로, 절대 외부(API)에 직접 노출해서는 안 됩니다.**

-   **문제점:**
    -   Entity의 모든 필드(password 등)가 원치 않게 외부에 노출될 수 있습니다.
    -   API 스펙이 데이터베이스 구조에 종속되어, DB 스키마 변경 시 API 스펙도 변경되는 문제가 발생합니다.

-   **해결책: DTO (데이터 전송 객체) 사용**
    계층 간 데이터를 주고받을 때 사용하는 별도의 깨끗한(Plain) Java 객체입니다.

    -   **`MemberPostDto`**: 회원 생성을 요청할 때 필요한 데이터(`email`, `name` 등)만 담는 객체
    -   **`MemberResponseDto`**: 클라이언트에게 응답으로 보내줄 데이터(`id`, `email`, `name` 등)만 담는 객체

**개선된 흐름:**
1.  **Controller**는 요청 본문(JSON)을 `MemberPostDto`로 받습니다.
2.  **Controller**는 `MemberPostDto`를 **Service**에 전달합니다.
3.  **Service**는 `MemberPostDto`를 `Member` Entity로 변환하여 **Repository**를 통해 저장합니다.
4.  **Service**는 저장된 `Member` Entity를 `MemberResponseDto`로 변환하여 **Controller**에 반환합니다.
5.  **Controller**는 `MemberResponseDto`를 JSON으로 변환하여 클라이언트에게 응답합니다.

이 구조는 각 계층의 역할을 더욱 명확하게 하고, API 스펙과 내부 데이터 모델을 분리하여 훨씬 유연하고 안정적인 애플리케이션을 만들게 해줍니다.

---

### **요약**

-   애플리케이션은 **Controller - Service - Repository**의 3계층으로 역할을 분리해야 합니다.
-   **Controller:** HTTP 요청/응답 처리 (관문)
-   **Service:** 핵심 비즈니스 로직 처리 (두뇌)
-   **Repository:** 데이터베이스 연동 처리 (저장소)
-   각 계층은 **의존성 주입(Dependency Injection)**을 통해 서로 연결됩니다.
-   계층 간 데이터 전송 시에는 Entity를 직접 사용하지 말고, **DTO(Data Transfer Object)를 사용**하여 역할을 명확히 분리하는 것이 매우 중요합니다.

### **출처 및 참고 자료**

-   **Baeldung - Spring a 3-Layer Application:**
    -   [https://www.baeldung.com/spring-layered-architecture](https://www.baeldung.com/spring-layered-architecture) (영어)
-   **우아한 테코 - DTO vs Entity:**
    -   [https://www.youtube.com/watch?v=J_Dr6R04i-4](https://www.youtube.com/watch?v=J_Dr6R04i-4) (영상)
