### **3-2. JPA와 Hibernate 개념 이해하기**

#### **1. 문제점: 왜 그냥 SQL을 쓰면 안 되나요?**

과거에는 Java 코드 안에 SQL 쿼리를 문자열 형태로 직접 작성했습니다.

```java
// 옛날 방식의 예시
String sql = "INSERT INTO user (username, email) VALUES (?, ?)";
// ... DB 연결, PreparedStatement 생성, 값 설정, 실행, 예외 처리 등등 ...
// ... 조회 시에는 ResultSet을 다시 User 객체로 변환하는 코드 ...
```

이 방식에는 여러 문제가 있습니다.

-   **반복적인 코드 (Boilerplate Code):** DB 연결, SQL 실행, 자원 해제 등 거의 모든 SQL 작업에 비슷한 코드가 반복적으로 들어갑니다.
-   **패러다임의 불일치:** Java는 객체지향 언어인데, 데이터는 관계형 데이터베이스에 저장됩니다. 개발자는 계속해서 객체 모델과 데이터 모델 사이에서 생각과 코드를 변환해야 합니다.
-   **SQL에 대한 의존성:** 특정 데이터베이스(예: MySQL)에 맞춰진 SQL을 작성하면, 나중에 다른 데이터베이스(예: PostgreSQL)로 교체할 때 많은 SQL 코드를 수정해야 할 수 있습니다.
-   **유지보수의 어려움:** 테이블에 컬럼이 하나 추가되면, 그 테이블을 사용하는 모든 SQL 쿼리와 객체 변환 코드를 일일이 찾아 수정해야 합니다.

#### **2. 해결책: ORM (Object-Relational Mapping)**

**ORM(객체-관계 매핑)**은 이러한 문제점을 해결하기 위해 등장한 기술입니다. 이름 그대로, **객체(Object)**와 **관계형 데이터베이스(Relational Database)**를 자동으로 **매핑(Mapping, 연결)**해주는 다리 역할을 합니다.

개발자는 더 이상 SQL 쿼리를 직접 작성하는 대신, 익숙한 Java 객체를 다루듯이 코드를 작성하면 됩니다. 그러면 ORM 프레임워크가 그 코드를 해석해서 적절한 SQL을 생성하고 실행한 뒤, 그 결과를 다시 Java 객체로 변환하여 돌려줍니다.

-   **비유:** ORM은 마치 '자동 번역기'와 같습니다. 개발자가 Java라는 언어로 "이 `User` 객체를 저장해줘"라고 말하면, ORM 번역기가 데이터베이스가 알아듣는 SQL 언어로 "INSERT INTO user..." 쿼리를 만들어 대신 실행해주는 것입니다.

#### **3. JPA (Java Persistence API)란?**

**JPA**는 ORM 기술을 사용하기 위한 **표준 명세(Standard Specification)**입니다. 즉, JPA는 실제 동작하는 코드가 아니라, "ORM 기술은 이렇게 만들어야 하고, 이런 기능들을 제공해야 한다"고 정해놓은 **인터페이스(규칙의 모음)**입니다.

-   **비유:** JPA는 'USB 포트 규격'과 같습니다. 세상에는 삼성, LG, Sandisk 등 여러 회사가 USB 메모리를 만들지만, 모두 이 USB 규격을 따르기 때문에 어떤 컴퓨터에 꽂아도 동작합니다. JPA도 마찬가지로, 여러 회사에서 JPA라는 표준 규칙을 따르는 ORM 프레임워크를 만듭니다.

#### **4. Hibernate란?**

**하이버네이트(Hibernate)**는 JPA라는 표준 명세를 **구현한 실제 구현체** 중 하나입니다. 즉, JPA라는 규칙에 맞춰 만들어진 가장 유명하고 널리 쓰이는 ORM 프레임워크 라이브러리입니다. Spring Boot는 JPA를 사용할 때 기본적으로 Hibernate를 사용하도록 설정되어 있습니다.

**정리:**
-   **ORM:** 객체와 DB 테이블을 매핑하는 기술
-   **JPA:** ORM을 위한 표준 명세(인터페이스)
-   **Hibernate:** JPA 명세를 구현한 실제 라이브러리(구현체)

우리는 JPA라는 '표준 방식'에 맞춰 코드를 작성하고, 그 코드는 내부적으로 Hibernate를 통해 실행되는 것입니다. 이렇게 하면 나중에 Hibernate가 아닌 다른 JPA 구현체로 바꾸더라도 코드 변경을 최소화할 수 있는 유연함을 얻게 됩니다.

#### **5. JPA 핵심 어노테이션 맛보기**

JPA를 사용하면 Java 클래스를 데이터베이스 테이블에 어떻게 매핑할까요? 바로 **어노테이션**을 사용합니다.

```java
import jakarta.persistence.*; // javax.persistence.* 일 수도 있습니다.

@Entity // "이 클래스는 데이터베이스 테이블과 매핑되는 클래스입니다" 라고 JPA에게 알려줌
public class Member {

    @Id // 이 필드가 테이블의 기본 키(PK)임을 명시
    @GeneratedValue(strategy = GenerationType.IDENTITY) // PK 값을 DB가 자동으로 생성(auto-increment)하도록 설정
    private Long id;

    @Column(name = "username") // 테이블의 'username' 컬럼과 매핑
    private String name;

    private String email; // @Column을 생략하면 필드명과 동일한 이름의 컬럼과 매핑

    // Getter, Setter, ...
}
```

이제 개발자는 `Member member = new Member();` 처럼 객체를 만들고 데이터를 설정한 뒤, JPA에게 `jpa.save(member);` 라고 요청하기만 하면 됩니다. `INSERT` SQL 쿼리는 Hibernate가 알아서 만들어 실행해줍니다.

---

### **요약**

-   과거에는 Java 코드에 SQL을 직접 작성하여 **반복적이고, 번거롭고, 유지보수가 어려웠습니다.**
-   **ORM**은 Java 객체와 데이터베이스 테이블을 자동으로 연결해주는 기술입니다.
-   **JPA**는 ORM 기술을 위한 **표준 인터페이스(규칙)**입니다.
-   **Hibernate**는 JPA라는 규칙을 구현한 **가장 인기 있는 실제 라이브러리**입니다.
-   우리는 `@Entity`, `@Id` 같은 JPA 표준 어노테이션을 사용하여 Java 클래스를 테이블에 매핑하고, SQL 없이 데이터 작업을 수행할 수 있습니다.

### **출처 및 참고 자료**

-   **자바 ORM 표준 JPA 프로그래밍 (김영한 저, 도서):**
    -   JPA를 가장 깊이 있게 이해할 수 있는 필독서입니다. [교보문고 링크](https://product.kyobobook.co.kr/detail/S000000935744)
-   **Baeldung - Introduction to JPA:**
    -   [https://www.baeldung.com/jpa-hibernate-guide](https://www.baeldung.com/jpa-hibernate-guide) (영어)
-   **우아한 테크 - 10분 테코톡: ORM과 JPA:**
    -   [https://www.youtube.com/watch?v=1npsG-y5b-k](https://www.youtube.com/watch?v=1npsG-y5b-k) (영상)
