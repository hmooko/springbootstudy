### **2-3. Spring Boot 프로젝트 생성하기 (feat. Spring Initializr)**

Spring Boot 프로젝트를 생성한다는 것은 단순히 빈 폴더를 만드는 것이 아니라, 애플리케이션이 동작하는 데 필요한 기본 구조, 설정 파일, 핵심 라이브러리(의존성)를 모두 갖춘 개발 환경을 구성하는 것을 의미합니다.

#### **1. Spring Initializr 접속하기**

먼저, 웹 브라우저를 열고 Spring Initializr 공식 사이트로 이동합니다.

- **URL:** [https://start.spring.io/](https://start.spring.io/)

이 사이트는 여러분이 만들고 싶은 Spring Boot 프로젝트의 명세를 입력받아, 그에 맞는 프로젝트 압축 파일을 생성해주는 곳입니다.

#### **2. 프로젝트 설정 상세 가이드**

사이트에 접속하면 여러 설정 옵션이 보입니다. 각 항목이 무엇을 의미하는지 자세히 알아봅시다.

| 항목                   | 설명                                                                                                                                                  | 추천 설정                          |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| **Project**          | 프로젝트의 빌드(Build) 및 의존성 관리 도구를 선택합니다. **Maven**과 **Gradle**이 있으며, 둘 다 훌륭한 도구입니다. 최근에는 간결한 스크립트와 빠른 속도 덕분에 Gradle의 인기가 높습니다.                           | **Gradle - Groovy**            |
| **Language**         | 프로젝트에서 사용할 프로그래밍 언어를 선택합니다.                                                                                                                         | **Java**                       |
| **Spring Boot**      | 사용할 Spring Boot의 버전을 선택합니다. 특별한 이유가 없다면, `(SNAPSHOT)`이나 `(M1)` 같은 표기가 없는 가장 최신의 안정화 버전(숫자만 있는 것)을 선택하는 것이 좋습니다.                                     | **3.x.x (가장 최신 안정화 버전)**       |
| **Project Metadata** | 프로젝트의 고유 정보를 설정합니다.                                                                                                                                 |                                |
| └ **Group**          | 보통 회사의 도메인을 거꾸로 적어 고유한 그룹명을 만듭니다. (예: `com.google`, `org.springframework`). 개인 프로젝트라면 `com.myproject` 와 같이 만들 수 있습니다. 이 설정은 Java 패키지의 기본 경로가 됩니다.   | `com.example`                  |
| └ **Artifact**       | 프로젝트의 결과물(jar 또는 war 파일)의 이름입니다. 즉, 프로젝트의 이름이라고 생각하면 됩니다.                                                                                           | `my-first-app`                 |
| └ **Name**           | 프로젝트의 이름입니다. 보통 Artifact와 동일하게 설정됩니다.                                                                                                               | `my-first-app`                 |
| └ **Description**    | 프로젝트에 대한 간단한 설명입니다.                                                                                                                                 | `My first Spring Boot project` |
| └ **Package name**   | `Group`과 `Artifact`를 조합하여 자동으로 생성되는 Java 소스코드의 최상위 패키지 이름입니다. (예: `com.example.myfirstapp`)                                                         | 자동 생성된 값 사용                    |
| **Packaging**        | 프로젝트를 어떤 형태로 포장할지 결정합니다. **Jar**는 내장 웹 서버(Tomcat 등)를 포함하여 단독으로 실행 가능한 파일 형식이며, **War**는 별도의 웹 서버에 배포할 때 사용합니다. 현대적인 마이크로서비스 아키텍처에서는 대부분 Jar를 사용합니다. | **Jar**                        |
| **Java**             | 사용할 Java의 버전을 선택합니다. 컴퓨터에 설치된 Java 버전과 맞추는 것이 좋으며, 보통 장기 지원(LTS) 버전인 **17** 또는 **21**을 권장합니다.                                                       | **17 또는 21**                   |

#### **3. 의존성(Dependencies) 추가하기**

프로젝트 설정의 핵심입니다. 'Dependencies'는 내 프로젝트가 어떤 기능들을 사용할 것인지를 명시하는 부분입니다. 'ADD DEPENDENCIES...' 버튼을 눌러 필요한 라이브러리를 추가할 수 있습니다.

처음 API를 만들기 위해 필요한 최소한의 의존성은 다음과 같습니다.

- **Spring Web**: RESTful API를 포함한 웹 애플리케이션을 만드는 데 필요한 모든 기능이 포함되어 있습니다. 내장 톰캣 서버, Spring MVC 프레임워크 등이 여기에 속합니다.
- **Lombok** (선택사항, 하지만 강력 추천): `@Getter`, `@Setter`, `@ToString` 등의 어노테이션으로 반복적인 Java 코드를 획기적으로 줄여주는 필수 라이브러리입니다.

#### **4. 프로젝트 생성 및 실행**

1.  모든 설정을 마쳤다면, 화면 하단의 **GENERATE** 버튼을 클릭합니다.
2.  설정한 내용에 따라 `.zip` 압축 파일이 다운로드됩니다.
3.  원하는 위치에 압축을 해제합니다.
4.  IntelliJ, Eclipse, VS Code 같은 IDE(통합 개발 환경)에서 'Open' 또는 'Import' 기능을 사용하여 압축 해제한 프로젝트 폴더를 엽니다.
    -   **IntelliJ Tip:** `File` > `Open...` 메뉴를 통해 프로젝트의 `build.gradle` 파일이나 프로젝트 폴더 자체를 선택하면 IDE가 알아서 Gradle 프로젝트로 인식하고 필요한 설정을 진행합니다.
5.  프로젝트 구조에서 `src/main/java/패키지명/프로젝트명Application.java` 파일을 찾을 수 있습니다. 이 파일이 Spring Boot 애플리케이션의 시작점입니다.
6.  IDE에서 이 Java 파일의 `main` 메소드 옆에 있는 실행 버튼(▶)을 누르면, 내장 서버가 동작하며 애플리케이션이 실행됩니다.

이제 여러분은 성공적으로 Spring Boot 프로젝트를 생성하고 실행한 것입니다! 물론 아직 아무 기능도 없지만, 웹 애플리케이션을 개발할 모든 준비가 끝난 상태입니다.

---

### **요약**

- **어디서?** Spring Initializr ([https://start.spring.io/](https://start.spring.io/)) 웹사이트에서
- **무엇을?** `Gradle`, `Java`, `Spring Boot` 안정 버전, `Jar` 패키징, `Java 17` 이상을 선택
- **핵심은?** `Dependencies`에서 `Spring Web`을 반드시 추가
- **어떻게?** `GENERATE` 버튼으로 다운로드 후, IDE에서 프로젝트 폴더를 열어 실행

### **출처 및 참고 자료**

- **공식 사이트:**
    - Spring Initializr: [https://start.spring.io/](https://start.spring.io/)
- **공식 문서:**
    - Spring Boot 공식 가이드 (RESTful 웹 서비스 빌드): [https://spring.io/guides/gs/rest-service/](https://spring.io/guides/gs/rest-service/)
- **추천 블로그:**
    - Baeldung - A Guide to Spring Initializr: [https://www.baeldung.com/spring-initializr](https://www.baeldung.com/spring-initializr) (영어)
    - 김영한님 인프런 로드맵 (프로젝트 생성 부분 참고): [https://www.inflearn.com/roadmaps/373](https://www.inflearn.com/roadmaps/373) (한국어, 유료 강의)
