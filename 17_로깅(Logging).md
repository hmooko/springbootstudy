# 5단계: 배포와 심화 학습 (Deployment & Beyond)

## 2. 로깅 (Logging)

애플리케이션을 개발하고 운영할 때, 프로그램 내부에서 어떤 일이 일어나고 있는지 기록을 남기는 행위를 **로깅**이라고 합니다. 로깅은 단순히 에러를 잡는 것뿐만 아니라, 시스템의 상태를 모니터링하고 사용자의 행동을 분석하는 데 필수적인 역할을 합니다.

Spring Boot는 **SLF4J**와 **Logback**이라는 강력한 로깅 프레임워크를 기본으로 채택하여, 개발자가 쉽고 효율적으로 로그를 관리할 수 있도록 지원합니다.

---

### **SLF4J와 Logback: 환상의 짝꿍**

-   **SLF4J (Simple Logging Facade for Java):** 로깅에 대한 **'표준 인터페이스(설계도)'**입니다. 개발자는 실제 로그를 처리하는 구현체가 무엇인지 신경 쓸 필요 없이, SLF4J가 제공하는 `Logger` 인터페이스에 맞춰 코드를 작성하면 됩니다. 이렇게 하면 나중에 다른 로깅 라이브러리로 쉽게 교체할 수 있는 유연한 구조가 됩니다.

-   **Logback:** SLF4J라는 '표준 인터페이스'를 실제로 구현한 **'로깅 구현체'**입니다. 과거에 많이 사용되던 Log4j의 후속 버전으로, 더 빠르고 강력한 성능과 다양한 설정 옵션을 제공합니다. Spring Boot의 기본 로깅 구현체입니다.

> **비유:** SLF4J는 'USB 포트'이고, Logback은 'USB 키보드'입니다. 어떤 USB 장치를 꽂아도 포트는 동일하듯, 개발자는 SLF4J라는 표준 포트에 맞춰 코딩하고, 실제 장치(Logback)는 Spring Boot가 알아서 연결해주는 것과 같습니다.

---

### **로그 레벨 (Log Level)**

로그 레벨은 로그의 심각도나 중요도를 나타내는 등급입니다. 설정된 레벨 이상의 로그만 출력됩니다. 예를 들어, `INFO` 레벨로 설정하면 `INFO`, `WARN`, `ERROR` 로그는 출력되지만, `DEBUG`와 `TRACE` 로그는 출력되지 않습니다.

**`ERROR > WARN > INFO > DEBUG > TRACE`** (오른쪽으로 갈수록 더 상세함)

-   **`ERROR`**: 요청을 처리하는 중 심각한 문제가 발생하여 기능이 정상 동작하지 않을 때 사용합니다. (예: 데이터베이스 연결 실패)
-   **`WARN`**: 지금 당장 에러는 아니지만, 향후 문제가 될 수 있는 잠재적 위험을 알릴 때 사용합니다. (예: 곧 지원 중단될 API 사용)
-   **`INFO`**: 애플리케이션의 주요 실행 흐름이나 상태 변경 등 의미 있는 정보를 나타낼 때 사용합니다. (Spring Boot의 기본 레벨)
-   **`DEBUG`**: 개발 단계에서 내부 상태나 변수 값을 확인하기 위한 상세 정보입니다.
-   **`TRACE`**: 가장 상세한 로그로, 특정 로직의 흐름을 하나하나 추적하고 싶을 때 사용합니다.

---

### **로그 사용법 및 설정**

#### **1. 코드에서 로그 사용하기**

Lombok 라이브러리의 `@Slf4j` 어노테이션을 사용하는 것이 가장 간편하고 일반적인 방법입니다.

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j // 이 어노테이션을 클래스에 붙여줍니다.
@Service
public class MyService {

    public void doSomething() {
        // log.info(), log.debug() 등을 바로 사용할 수 있습니다.
        log.info("Service 로직 시작");

        try {
            // ... 비즈니스 로직 ...
            log.debug("중간 데이터 확인: {}", someData);
        } catch (Exception e) {
            log.error("로직 수행 중 예외 발생", e); // 예외 객체를 함께 넘겨주면 스택 트레이스가 기록됩니다.
        }
    }
}
```
- `@Slf4j`는 컴파일 시점에 `private static final Logger log = LoggerFactory.getLogger(MyService.class);` 코드를 자동으로 생성해줍니다.
- `{}`를 사용한 포맷팅은 문자열을 미리 더하지 않고, 로그 레벨이 활성화되었을 때만 문자열을 조합하므로 성능상 이점이 있습니다.

#### **2. `application.properties`로 간단하게 설정하기**

`src/main/resources/application.properties` 파일에서 간단하게 로그 설정을 변경할 수 있습니다.

```properties
# 애플리케이션 전체의 기본 로그 레벨을 DEBUG로 변경
logging.level.root=DEBUG

# 특정 패키지(com.example.demo.service) 하위는 INFO 레벨로 설정
logging.level.com.example.demo.service=INFO

# 특정 파일 이름으로 로그를 저장 (콘솔에도 계속 출력됨)
logging.file.name=logs/my-app.log

# 로그 파일의 크기가 10MB에 도달하면 파일을 분리(archive)하고, 총 100MB까지만 보관
logging.file.max-size=10MB
logging.file.total-size-cap=100MB

# 콘솔에 출력되는 로그 패턴 변경
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
```

#### **3. `logback-spring.xml`로 상세하게 설정하기**

환경별(개발, 운영)로 다른 로그 설정을 적용하거나, 특정 로그는 파일에만 저장하는 등 복잡한 설정이 필요할 때는 `src/main/resources` 폴더에 `logback-spring.xml` 파일을 만들어 사용합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/my-app.log</file>
        <encoder>
            <pattern>%d %-5level [%thread] %logger{36}: %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/archived/my-app.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>

    <!-- 운영 환경(prod)일 때만 FILE appender를 사용 -->
    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="FILE" />
        </root>
    </springProfile>

    <!-- 개발 환경(dev)일 때는 CONSOLE appender만 사용 -->
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>
</configuration>
```

---

### **요약**

-   **로깅**은 디버깅, 모니터링, 분석을 위한 필수적인 활동입니다.
-   Spring Boot는 **SLF4J(인터페이스) + Logback(구현체)** 조합을 기본 로깅 시스템으로 사용합니다.
-   로그 레벨(**ERROR, WARN, INFO, DEBUG, TRACE**)을 통해 출력되는 로그의 상세도를 조절할 수 있습니다.
-   간단한 설정은 `application.properties`에서, 복잡하고 환경별로 다른 설정은 `logback-spring.xml`에서 관리하는 것이 효율적입니다.

---

### **출처 및 참고 자료**

-   **공식 문서:**
    -   Spring Boot Logging: [https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging)
    -   Logback Project: [https://logback.qos.ch/](https://logback.qos.ch/)
-   **개념 이해를 위한 블로그:**
    -   Spring Boot Logging (Baeldung): [https://www.baeldung.com/spring-boot-logging](https://www.baeldung.com/spring-boot-logging)
    -   [Spring] Logback 사용법 및 설정 (velog): [https://velog.io/@sihyung92/Spring-Logback-사용법-및-설정-정리](https://velog.io/@sihyung92/Spring-Logback-%EC%82%AC%EC%9A%A9%EB%B2%95-%EB%B0%8F-%EC%84%A4%EC%A%95-%EC%A0%95%EB%A6%AC)
