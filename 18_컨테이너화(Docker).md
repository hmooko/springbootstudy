# 5단계: 배포와 심화 학습 (Deployment & Beyond)

## 3. 컨테이너화 (Containerization) with Docker

내가 만든 애플리케이션을 다른 사람의 컴퓨터나 서버에서 실행하려면, 내 컴퓨터에 설치된 모든 프로그램(Java, 데이터베이스 등)을 똑같이 설치해야 하는 번거로움이 있습니다. **컨테이너화**는 이러한 문제를 해결하기 위해 애플리케이션과 그 실행에 필요한 모든 환경(라이브러리, 설정 등)을 하나로 묶어, 어디서든 동일하게 실행될 수 있는 독립적인 패키지로 만드는 기술입니다. **Docker**는 현재 가장 널리 사용되는 컨테이너 기술입니다.

---

### **Docker의 핵심 개념**

-   **이미지 (Image):** 컨테이너를 생성하기 위한 **읽기 전용 템플릿(설계도)**입니다. 이미지에는 애플리케이션 코드, 실행 환경(JRE), 라이브러리 등 컨테이너 실행에 필요한 모든 것이 포함되어 있습니다.
-   **컨테이너 (Container):** Docker 이미지를 실행한 **실제 인스턴스(제품)**입니다. 하나의 이미지로부터 여러 개의 컨테이너를 생성할 수 있으며, 각 컨테이너는 격리된 환경에서 독립적으로 실행됩니다.
-   **Dockerfile:** Docker 이미지를 어떻게 만들지를 정의하는 **스크립트 파일(레시피)**입니다. 개발자는 이 파일에 필요한 명령어를 순서대로 작성하여 이미지를 빌드합니다.

> **비유:** Docker 이미지는 '붕어빵 틀', 컨테이너는 그 틀로 찍어낸 '붕어빵', Dockerfile은 '붕어빵을 만드는 레시피'라고 생각할 수 있습니다.

---

### **Spring Boot 애플리케이션 Dockerize 하기 (Step-by-Step)**

#### **1단계: 애플리케이션을 JAR 파일로 빌드하기**

먼저 Spring Boot 애플리케이션을 실행 가능한 JAR 파일로 만들어야 합니다. 프로젝트의 루트 디렉터리에서 다음 명령어를 실행합니다.

-   **Gradle 사용 시:**
    ```bash
    ./gradlew build
    ```
-   **Maven 사용 시:**
    ```bash
    ./mvnw clean package
    ```

빌드가 성공하면 Gradle은 `build/libs/`, Maven은 `target/` 디렉터리에 `프로젝트명-0.0.1-SNAPSHOT.jar`와 같은 파일이 생성됩니다.

#### **2단계: `Dockerfile` 작성하기**

프로젝트 루트 디렉터리에 `Dockerfile`이라는 이름의 파일을 만들고 다음 내용을 작성합니다.

```dockerfile
# 1. 베이스 이미지 선택 (Java 17 실행 환경)
# 가볍고 보안에 유리한 Alpine Linux 기반의 이미지를 사용합니다.
FROM openjdk:17-jdk-alpine

# 2. 컨테이너 내부의 작업 디렉터리 설정
WORKDIR /app

# 3. 빌드된 JAR 파일을 컨테이너 내부로 복사
# Gradle 기준 경로입니다. Maven을 사용했다면 target/*.jar로 변경하세요.
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar

# 4. 컨테이너가 외부에 노출할 포트 지정
# Spring Boot의 기본 포트인 8080을 노출합니다.
EXPOSE 8080

# 5. 컨테이너가 시작될 때 실행할 명령어
# "java -jar app.jar" 명령어로 애플리케이션을 실행합니다.
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### **3단계: Docker 이미지 빌드하기**

작성한 `Dockerfile`을 사용하여 Docker 이미지를 빌드합니다. 터미널에서 다음 명령어를 실행합니다.

```bash
# -t 옵션으로 이미지의 이름과 태그(버전)를 지정합니다. '.'은 현재 디렉터리의 Dockerfile을 사용하라는 의미입니다.
docker build -t my-spring-app:0.1 .
```

빌드가 완료되면 `docker images` 명령어로 내 컴퓨터에 생성된 이미지를 확인할 수 있습니다.

#### **4. Docker 컨테이너 실행하기**

빌드한 이미지를 사용하여 컨테이너를 실행합니다.

```bash
# -p 옵션으로 호스트 컴퓨터의 포트와 컨테이너의 포트를 연결합니다.
# 형식: -p [호스트 포트]:[컨테이너 포트]
docker run -p 8080:8080 my-spring-app:0.1
```

이제 웹 브라우저나 API 테스트 도구에서 `http://localhost:8080`으로 접속하면, 내 컴퓨터가 아닌 Docker 컨테이너 안에서 실행되고 있는 Spring Boot 애플리케이션에 접근할 수 있습니다.

---

### **요약**

-   **Docker**는 애플리케이션을 실행 환경과 함께 패키징하여, 어디서든 동일하게 실행되도록 하는 **컨테이너 기술**입니다.
-   **`Dockerfile`**에 컨테이너 생성 방법을 정의하고(`FROM`, `COPY`, `ENTRYPOINT` 등), `docker build` 명령어로 **이미지**를 만듭니다.
-   `docker run` 명령어로 이미지를 **컨테이너**로 실행하며, `-p` 옵션으로 포트를 연결하여 외부에서 접근할 수 있게 합니다.
-   이 과정을 통해 "내 컴퓨터에서는 되는데, 서버에서는 안 돼요"와 같은 고질적인 문제를 해결할 수 있습니다.

---

### **출처 및 참고 자료**

-   **공식 문서:**
    -   Spring Boot with Docker Guide: [https://spring.io/guides/gs/spring-boot-docker/](https://spring.io/guides/gs/spring-boot-docker/)
    -   Dockerizing a Spring Boot Application: [https://docs.docker.com/language/java/spring-boot/](https://docs.docker.com/language/java/spring-boot/)
-   **개념 이해를 위한 블로그:**
    -   Dockerizing a Spring Boot Application (Baeldung): [https://www.baeldung.com/spring-boot-docker-images](https://www.baeldung.com/spring-boot-docker-images)
    -   초보를 위한 도커 안내서 (subicura): [https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)
