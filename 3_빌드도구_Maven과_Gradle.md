# 빌드 도구: Maven vs Gradle

안녕하세요! 스프링 부트(Spring Boot)와 백엔드 개발에 첫발을 내디디신 것을 진심으로 환영합니다.

스프링 부트로 프로젝트를 시작할 때 가장 먼저 마주하게 되는 선택 중 하나가 바로 **빌드 도구(Build Tool)** 를 `Maven(메이븐)`으로 할 것인지, `Gradle(그래들)`로 할 것인지 정하는 것입니다. 두 도구의 차이점을 아주 세세하고 꼼꼼하게 알려드릴게요.

### 1. 빌드 도구(Build Tool)란 무엇인가요?

먼저 빌드 도구가 무엇인지부터 알아야겠죠?

개발자가 작성한 자바 코드(.java 파일)와 여러 설정 파일(.xml, .properties 등)은 컴퓨터나 서버가 바로 실행할 수 없습니다. 이것들을 JVM(자바 가상 머신)이나 WAS(웹 애플리케이션 서버)가 이해할 수 있는 형태(주로 .jar 또는 .war 파일)로 변환하고 포장하는 과정이 필요한데, 이 과정을 **빌드(Build)** 라고 합니다.

빌드 도구는 이 빌드 과정을 자동화해주는 프로그램입니다. 단순히 코드 변환뿐만 아니라 다음과 같이 프로젝트에 필요한 복잡한 작업들을 대신 처리해 줍니다.

*   **의존성 관리(Dependency Management):** 프로젝트에 필요한 외부 라이브러리(예: Spring, Lombok 등)를 설정 파일에 이름과 버전만 적어주면, 인터넷을 통해 자동으로 다운로드하고 관리해 줍니다.
*   **컴파일 및 패키징:** 소스 코드를 컴파일하고 실행 가능한 파일로 묶어줍니다.
*   **테스트 실행:** 작성된 테스트 코드를 실행하고 결과를 알려줍니다.
*   **배포:** 완성된 애플리케이션을 서버에 배포하는 작업을 수행합니다.

과거에는 이런 작업들을 개발자가 수동으로 해야 했지만, 이제는 Maven이나 Gradle 같은 빌드 도구 덕분에 아주 편리해졌습니다.

---

### 2. Maven (메이븐)

Maven은 아파치(Apache) 재단에서 만든 자바 프로젝트 관리 도구입니다. 오랫동안 자바 진영의 표준 빌드 도구로 사용되어 왔으며, 지금도 수많은 프로젝트에서 사용되고 있습니다.

#### 주요 특징

*   **`pom.xml` 기반 설정:** Maven의 모든 설정은 `pom.xml` (Project Object Model)이라는 XML 형식의 파일 하나로 관리됩니다. 이 파일에 프로젝트 정보, 사용할 라이브러리, 플러그인 등을 명시합니다.
*   **컨벤션 중심(Convention over Configuration):** "설정보다 관습"이라는 철학을 따릅니다. 정해진 프로젝트 구조와 규칙(예: 소스 코드는 `src/main/java`에 위치)을 따르면 복잡한 설정 없이도 쉽게 프로젝트를 구성할 수 있습니다.
*   **안정성과 방대한 자료:** 오래된 만큼 매우 안정적이고, 관련 문서나 커뮤니티의 자료가 풍부하여 문제가 생겼을 때 해결책을 찾기 쉽습니다.

#### `pom.xml` 예시 (Spring Boot)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

</project>
```

---

### 3. Gradle (그래들)

Gradle은 Maven 이후에 등장한 더 현대적인 빌드 도구입니다. Maven의 장점은 계승하고 단점은 보완하는 것을 목표로 만들어졌으며, 특히 안드로이드 앱의 공식 빌드 시스템으로 채택되면서 점유율이 크게 높아졌습니다.

#### 주요 특징

*   **`build.gradle` 기반 설정:** Gradle은 `Groovy` 또는 `Kotlin`이라는 스크립트 언어를 사용하여 설정을 관리합니다. 파일 이름은 `build.gradle`(Groovy 사용 시) 또는 `build.gradle.kts`(Kotlin 사용 시)입니다.
*   **유연성과 간결함:** XML에 비해 스크립트 기반의 설정은 훨씬 간결하고 가독성이 높습니다. 또한 프로그래밍 코드를 사용하므로 복잡하고 동적인 빌드 로직을 쉽게 작성할 수 있습니다.
*   **뛰어난 성능:** Gradle은 빌드 캐시(Cache), 점진적 빌드(Incremental Build) 등 다양한 기술을 사용하여 Maven보다 훨씬 빠른 빌드 속도를 보여줍니다. 프로젝트 규모가 클수록 성능 차이가 두드러집니다.

#### `build.gradle` (Groovy DSL) 예시 (Spring Boot)

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.2'
    id 'io.spring.dependency-management' version '1.1.5'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
}
```
XML 방식의 Maven보다 훨씬 코드가 짧고 간결한 것을 볼 수 있습니다.

---

### 4. Maven vs Gradle 핵심 차이점 비교

| 구분 | Maven (메이븐) | Gradle (그래들) |
| :--- | :--- | :--- |
| **설정 파일** | `pom.xml` (XML) | `build.gradle` (Groovy/Kotlin DSL) |
| **가독성/간결성** | 장황하고 가독성이 떨어짐 | 간결하고 가독성이 높음 |
| **빌드 속도** | 상대적으로 느림 | 캐시 사용 등으로 매우 빠름 (최대 100배) |
| **유연성** | 정해진 규칙을 따르는 구조, 유연성 부족 | 스크립트 기반으로 매우 유연하고 자유로운 구성 가능 |
| **학습 곡선** | XML 구조만 이해하면 되어 비교적 쉬움 | Groovy/Kotlin 문법 학습이 필요하여 초기 진입 장벽이 있음 |
| **멀티 프로젝트** | 상속 구조로 관리하여 복잡해질 수 있음 | 설정 주입 방식으로 멀티 프로젝트 관리에 더 적합함 |
| **생태계/점유율** | 전통적으로 점유율이 높고 안정적 | 최신 프로젝트와 안드로이드 진영을 중심으로 점유율이 빠르게 상승 중 |

### 5. 그래서 초보자는 무엇을 선택해야 할까요?

**결론부터 말하면, 현재 시점에서는 `Gradle`로 시작하는 것을 조금 더 추천합니다.**

*   **이유 1: 성능과 생산성**
    빠른 빌드 속도는 개발 과정에서 스트레스를 줄여주고 생산성을 높여줍니다. 간결한 설정 파일은 유지보수를 더 쉽게 만듭니다.

*   **이유 2: 최신 트렌드**
    많은 새로운 스프링 부트 프로젝트들이 Gradle을 채택하고 있으며, 앞으로의 발전 가능성도 더 높게 평가받고 있습니다.

*   **이유 3: 유연성**
    프로젝트가 복잡해질수록 Gradle의 유연한 구조가 큰 장점으로 다가올 것입니다.

**하지만 `Maven`이 나쁜 선택이라는 의미는 절대 아닙니다.**

*   안정적이고 방대한 레퍼런스를 바탕으로 한 쉬운 문제 해결 능력은 초보자에게 큰 장점입니다.
*   많은 기업과 오픈소스 프로젝트가 여전히 Maven을 표준으로 사용하고 있으므로, Maven을 다룰 줄 아는 것은 중요합니다.

**최종 제안:**
1.  **`Gradle`을 기본으로 선택하여 학습을 진행해 보세요.** 최신 기술 트렌드와 성능의 이점을 누릴 수 있습니다.
2.  **`Maven` 프로젝트를 접할 기회가 생기면 겁내지 마세요.** `pom.xml`의 구조는 직관적이어서 Gradle을 이해했다면 금방 적응할 수 있습니다.

스프링 부트 프로젝트를 생성해주는 `start.spring.io` 사이트에서도 Maven과 Gradle을 모두 지원하므로, 언제든지 원하는 빌드 도구로 프로젝트를 시작할 수 있습니다.

---

### 6. 요약 및 참고 자료

#### 요약

*   **빌드 도구**는 소스 코드를 실행 가능한 파일로 만들고, 라이브러리 관리 등을 자동화하는 필수 도구입니다.
*   **Maven**은 XML 기반(`pom.xml`)으로, 안정적이고 자료가 많지만 설정이 길고 빌드가 느립니다.
*   **Gradle**은 스크립트 기반(`build.gradle`)으로, 간결하고 유연하며 빌드 속도가 매우 빠르지만 초기 학습 곡선이 조금 있습니다.
*   **초보자에게는** 최신 트렌드와 성능을 고려하여 **Gradle을 추천**하지만, Maven 역시 여전히 중요한 도구입니다.

#### 도움이 될 만한 사이트 및 블로그

*   **공식 문서:**
    *   [Apache Maven Project](https://maven.apache.org/)
    *   [Gradle Official Website](https://gradle.org/)
*   **비교 블로그:**
    *   [Spring: Maven과 Gradle의 차이 - 티스토리](https://dev-code-note.tistory.com/12)
    *   [메이븐(Maven)과 그래들(Gradle)의 개념 및 비교 - 티스토리](https://siyoon210.tistory.com/140)
    *   [Spring Boot 빌드 관리 도구 Maven과 Gradle 비교하기 - 티스토리](https://jisooo.tistory.com/100)
