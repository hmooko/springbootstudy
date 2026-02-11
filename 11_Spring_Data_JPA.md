### **3-3. Spring Data JPA**

#### **1. JPA만으로는 부족한가?**

JPA를 사용하면 객체 중심으로 데이터베이스를 다룰 수 있게 되지만, 데이터를 조작하기 위해서는 여전히 `EntityManager`라는 것을 주입받아 코드를 작성해야 합니다.

```java
// 순수 JPA를 사용한 데이터 저장 계층(Repository) 예시
@Repository
public class MemberJpaRepository {

    @PersistenceContext
    private EntityManager em;

    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    public Member find(Long id) {
        return em.find(Member.class, id);
    }
    // ... findAll(), delete() 등등의 메소드 구현 필요 ...
}
```

위 코드처럼, 기본적인 저장(save), 조회(find), 삭제(delete) 같은 기능조차도 모든 엔티티(Entity)마다 비슷한 코드를 반복해서 작성해야 하는 번거로움이 남습니다. 개발자들은 이 반복적인 작업을 '보일러플레이트 코드(Boilerplate Code)'라고 부릅니다.

#### **2. Spring Data JPA: 반복의 종말**

**Spring Data JPA**는 이 지루한 반복 작업을 완전히 없애기 위해 등장했습니다. Spring Data 프로젝트의 일부로서, 데이터 접근 계층(Repository)을 개발하는 과정을 극도로 단순화시켜 줍니다.

핵심 아이디어는 **"인터페이스만 만들면, 구현은 Spring이 알아서 해줄게!"** 입니다.

#### **3. 마법의 `JpaRepository` 인터페이스**

Spring Data JPA를 사용하는 방법은 놀라울 정도로 간단합니다.

1.  `interface`를 하나 만듭니다.
2.  Spring Data JPA가 제공하는 `JpaRepository` 인터페이스를 `extends` (상속) 받습니다.

```java
import org.springframework.data.jpa.repository.JpaRepository;

// 이것이 전부입니다!
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

-   `JpaRepository<Member, Long>` 에서
    -   첫 번째 인자 `Member`는 이 리포지토리가 다룰 엔티티 클래스를 의미합니다.
    -   두 번째 인자 `Long`은 해당 엔티티의 기본 키(PK) 필드의 데이터 타입을 의미합니다. (`Member` 엔티티의 `id` 필드 타입)

단지 이렇게 인터페이스를 선언하는 것만으로, Spring Data JPA는 실행 시점에 이 인터페이스를 구현한 클래스를 **자동으로 생성**하여 Spring 컨테이너에 등록해줍니다. 그리고 그 구현체에는 아래와 같은 핵심적인 CRUD 기능들이 모두 포함되어 있습니다.

-   `save(S entity)`: 저장 및 수정 (INSERT/UPDATE)
-   `findById(ID id)`: PK로 한 건 조회 (SELECT)
-   `findAll()`: 모든 데이터 조회 (SELECT)
-   `deleteById(ID id)`: PK로 데이터 삭제 (DELETE)
-   `count()`: 데이터의 총 개수 조회 (SELECT COUNT)
-   ... 등등 수많은 기본 기능 제공

이제 서비스 계층에서는 이 `MemberRepository`를 주입받아 바로 `memberRepository.save(member)` 와 같이 메소드를 호출할 수 있습니다. 구현 클래스를 만들지 않았는데도 말이죠!

#### **4. 쿼리 메소드 (Query Methods)**

기본적인 CRUD 외에, '이메일로 회원 찾기'나 '이름으로 회원 찾기' 같은 특정 조건의 쿼리가 필요할 때는 어떻게 할까요?

Spring Data JPA는 **메소드 이름을 분석하여 자동으로 쿼리를 생성**하는 경이로운 '쿼리 메소드' 기능을 제공합니다.

```java
import java.util.List;
import java.util.Optional;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MemberRepository extends JpaRepository<Member, Long> {

    // 이메일 주소로 회원을 찾는 쿼리를 자동으로 생성
    // SELECT m FROM Member m WHERE m.email = ?
    Optional<Member> findByEmail(String email);

    // 특정 이름으로 시작하는 회원을 찾는 쿼리를 자동으로 생성
    // SELECT m FROM Member m WHERE m.name LIKE ?%
    List<Member> findByNameStartingWith(String namePrefix);
}
```

정해진 규칙(예: `findBy[필드명]`, `countBy[필드명]`, `...And...`, `...Or...`, `...Like`)에 따라 메소드 이름만 지어주면, Spring Data JPA가 그 이름에 맞는 JPQL(JPA 쿼리 언어)을 생성해서 실행합니다. 더 이상 간단한 쿼리를 위해 `@Query` 어노테이션으로 JPQL을 작성할 필요가 없습니다.

#### **5. MVC 패턴과 Repository 계층**

이러한 방식은 애플리케이션을 계층으로 나누는 **MVC(Model-View-Controller) 패턴**과 완벽하게 들어맞습니다.

-   **Controller:** HTTP 요청을 받고 응답을 보냅니다.
-   **Service:** 비즈니스 로직을 처리합니다. Controller로부터 요청을 위임받고, Repository를 호출하여 데이터를 조작합니다.
-   **Repository:** 데이터베이스 접근을 담당합니다. Spring Data JPA를 통해 만든 인터페이스가 이 계층에 해당합니다.

이 구조를 통해 각 계층은 자신의 역할에만 충실하게 되어, 코드가 훨씬 깔끔해지고 유지보수가 쉬워집니다.

---

### **요약**

-   **Spring Data JPA**는 JPA를 더 쉽고 간편하게 사용하기 위한 기술입니다.
-   `JpaRepository` 인터페이스를 상속받는 **인터페이스를 선언**하는 것만으로, 기본적인 CRUD 기능이 자동으로 구현됩니다.
-   **쿼리 메소드** 기능을 통해, 정해진 규칙에 따라 메소드 이름만 작성하면 원하는 조회 쿼리를 자동으로 생성할 수 있습니다.
-   이를 통해 데이터 접근 계층(Repository)의 코드를 획기적으로 줄여, 개발자는 **비즈니스 로직(Service)에 더 집중**할 수 있습니다.

### **출처 및 참고 자료**

-   **Spring Data JPA 공식 문서:**
    -   [https://docs.spring.io/spring-data/jpa/reference/repositories.html](https://docs.spring.io/spring-data/jpa/reference/repositories.html)
-   **Baeldung - Introduction to Spring Data JPA:**
    -   [https://www.baeldung.com/spring-data-jpa-tutorial](https://www.baeldung.com/spring-data-jpa-tutorial) (영어)
-   **김영한님 - 스프링 데이터 JPA (인프런 강의):**
    -   Spring Data JPA를 깊이 있게 배우고 싶다면 강력 추천하는 강의입니다. [강의 링크](https://www.inflearn.com/course/스프링-데이터-jpa)
