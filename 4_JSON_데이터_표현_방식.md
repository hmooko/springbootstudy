# JSON (JavaScript Object Notation) 이해하기

백엔드 기초 다지기 1단계의 마지막 주제, JSON에 오신 것을 환영합니다. API를 통해 데이터를 주고받을 때, 어떤 "언어"와 "형식"으로 대화할지 정해야 하는데, 현재 웹에서 가장 널리 사용되는 표준이 바로 JSON입니다.

### 1. JSON이란 무엇인가요?

**JSON(JavaScript Object Notation)**은 이름에서 알 수 있듯, 원래 자바스크립트(JavaScript)의 객체를 표기하는 방법에서 시작된 **데이터 교환 형식**입니다. 하지만 지금은 자바스크립트에 종속되지 않고, 거의 모든 프로그래밍 언어에서 사용할 수 있는 독립적인 표준이 되었습니다.

JSON의 가장 큰 특징은 다음과 같습니다.

*   **가볍다 (Lightweight):** 불필요한 태그 없이 순수한 텍스트로만 이루어져 있어 용량이 매우 작습니다.
*   **인간이 읽고 쓰기 쉽다 (Human-readable):** 문법이 단순하고 직관적이어서 사람이 눈으로 보고 쉽게 이해할 수 있습니다.
*   **기계가 분석하고 생성하기 쉽다 (Machine-parseable):** 대부분의 언어에서 JSON을 분석(Parsing)하여 객체로 변환하거나, 객체를 JSON 문자열로 변환하는 라이브러리를 기본적으로 제공합니다.

이러한 장점 덕분에, RESTful API에서 서버와 클라이언트가 데이터를 주고받을 때 XML을 대체하여 사실상의 표준(de facto standard)으로 자리 잡았습니다.

---

### 2. JSON의 기본 구조와 문법

JSON은 **키(Key)-값(Value) 쌍**으로 이루어져 있습니다. 이 기본 구조만 알면 절반은 이해한 것입니다.

*   **키(Key):** 반드시 **큰따옴표("")**로 감싼 문자열이어야 합니다.
*   **값(Value):** 다양한 데이터 타입을 가질 수 있습니다.
    1.  **문자열(String):** 큰따옴표("")로 감싼 텍스트 (예: `"안녕하세요"`)
    2.  **숫자(Number):** 정수 또는 실수 (예: `100`, `3.14`)
    3.  **불리언(Boolean):** `true` 또는 `false`
    4.  **객체(Object):** 다른 JSON 객체. 중괄호`{}`로 감쌉니다.
    5.  **배열(Array):** 값의 순서 있는 목록. 대괄호`[]`로 감쌉니다.
    6.  **null:** 값이 없음을 나타내는 특별한 값.

*   키-값 쌍들은 쉼표(`,`)로 구분됩니다.
*   전체 JSON 데이터는 하나의 객체(`{}`) 또는 배열(`[]`)로 감싸여야 합니다.

#### JSON 예시

`교육과정.md`의 미니 프로젝트 #1에서 구상했던 "좋아하는 영화 정보"를 JSON으로 표현하면 다음과 같습니다.

```json
{
  "title": "인셉션",
  "englishTitle": "Inception",
  "releaseYear": 2010,
  "isOscarWinner": true,
  "rating": 9.5,
  "director": {
    "name": "크리스토퍼 놀란",
    "birthYear": 1970
  },
  "genres": [
    "SF",
    "액션",
    "스릴러"
  ],
  "mainActors": null
}
```

*   `"title"`, `"releaseYear"`, `"isOscarWinner"` 등은 모두 **키(Key)**입니다.
*   `"인셉션"`은 **문자열**, `2010`은 **숫자**, `true`는 **불리언** 값입니다.
*   `"director"`의 값은 또 다른 **객체(Object)**입니다.
*   `"genres"`의 값은 여러 문자열을 담고 있는 **배열(Array)**입니다.
*   `"mainActors"`의 값은 **null**입니다.

---

### 3. Spring Boot와 JSON의 관계

Spring Boot로 백엔드 개발을 할 때 JSON이 중요한 이유는, **개발자가 직접 JSON을 다룰 일이 거의 없기 때문**입니다. 스프링 부트는 이 변환 과정을 자동으로 처리해 줍니다.

*   **직렬화 (Serialization):** 서버가 클라이언트에게 데이터를 보낼 때, **Java 객체**를 **JSON 문자열**로 변환하는 과정입니다.
    *   `@RestController`가 붙은 컨트롤러에서 Java 객체(예: `MovieDto`)를 반환하면, 스프링 부트가 내부적으로 **Jackson**이라는 라이브러리를 사용해 JSON으로 자동 변환 후 응답합니다.

*   **역직렬화 (Deserialization):** 클라이언트가 서버에게 데이터를 보낼 때, 요청 본문(Request Body)에 담긴 **JSON 문자열**을 **Java 객체**로 변환하는 과정입니다.
    *   클라이언트가 보낸 JSON 데이터를 컨트롤러의 메서드에서 `@RequestBody` 어노테이션이 붙은 Java 객체(예: `MovieDto`)로 받으면, 스프링 부트가 알아서 해당 객체로 변환해 줍니다.

이 자동 변환 덕분에 개발자는 Java 객체를 다루는 데만 집중할 수 있어 매우 편리합니다.

---

### 요약

*   **JSON**은 서버와 클라이언트 간에 데이터를 교환하기 위한 **가볍고 효율적인 텍스트 형식**입니다.
*   **키-값 쌍**을 기본 구조로 하며, 문자열, 숫자, 불리언, 객체, 배열, null 등 다양한 데이터 타입을 표현할 수 있습니다.
*   **Spring Boot**에서는 **Jackson** 라이브러리를 통해 Java 객체와 JSON 간의 변환(직렬화/역직렬화)을 **자동으로 처리**해주므로 개발이 매우 편리합니다.

### 도움이 될 만한 자료

*   **JSON 공식 소개 (한국어):** [https://www.json.org/json-ko.html](https://www.json.org/json-ko.html)
*   **MDN - JSON 소개:** [https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/JSON](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/JSON)
*   **JSON 데이터를 검증하고 예쁘게 정렬해주는 사이트:** [https://jsonlint.com/](https://jsonlint.com/)
