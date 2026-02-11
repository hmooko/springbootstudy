# API와 RESTful API 완벽 정복 (초보자용)

안녕하세요! 백엔드 개발 공부를 시작하신 것을 진심으로 응원합니다. API와 RESTful API는 백엔드 개발의 핵심적인 개념이므로 확실히 알아두시면 앞으로의 학습에 큰 도움이 될 것입니다. 최대한 이해하기 쉽게, 그리고 아주 자세히 설명해 드릴게요.

### 1. API란 무엇일까요? (Application Programming Interface)

API를 처음 접할 때 가장 많이 사용하는 비유는 **레스토랑의 점원(웨이터)**입니다.

손님(클라이언트)이 레스토랑에 가서 요리를 주문한다고 상상해 보세요.
*   **손님 (Client)**: "파스타 주세요!" 라고 요청하는 프로그램 (예: 웹 브라우저, 스마트폰 앱)
*   **주방 (Server)**: 요청받은 파스타를 실제로 만드는 곳 (예: 데이터를 가진 서버 컴퓨터)
*   **점원 (API)**: 손님의 주문을 받아서 주방에 전달하고, 완성된 요리를 다시 손님에게 가져다주는 역할

이 과정에서 손님은 주방의 복잡한 레시피나 요리 과정을 전혀 알 필요가 없습니다. 그저 메뉴판(정해진 주문 방식)을 보고 점원에게 주문하면 됩니다. 점원은 손님과 주방 사이의 약속된 방법으로 소통하며 중간 다리 역할을 합니다.

**API(Application Programming Interface)**가 바로 이 '점원'의 역할을 합니다. 즉, **프로그램(Application)들이 서로 소통하기 위해 정해놓은 약속(Interface)**이라고 할 수 있습니다.

*   **Application**: 우리가 사용하는 앱, 웹사이트 등 모든 소프트웨어
*   **Programming**: 컴퓨터 프로그램
*   **Interface**: 두 개의 다른 장치나 프로그램이 서로 만나서 소통하는 접점

예를 들어, 우리가 사용하는 '날씨 앱'은 자체적으로 전 세계의 날씨 데이터를 모두 가지고 있지 않습니다. 대신, '기상청'에서 제공하는 날씨 데이터 API를 이용합니다.

1.  **날씨 앱 (클라이언트)**이 "서울의 현재 날씨 알려줘" 라고 **기상청 API (점원)**에 요청을 보냅니다.
2.  **기상청 API**는 이 요청을 받아 **기상청 서버 (주방)**에 전달합니다.
3.  **기상청 서버**는 "서울 날씨: 맑음, 25도" 라는 데이터를 준비해서 **API**에 전달합니다.
4.  **API**는 이 데이터를 다시 **날씨 앱**에 전달해주고, 우리는 앱 화면에서 날씨 정보를 볼 수 있습니다.

이처럼 API는 복잡한 내부 구현을 몰라도, 정해진 규칙에 따라 요청하고 응답을 받음으로써 다른 프로그램의 기능이나 데이터를 쉽게 사용할 수 있게 해주는 매우 중요한 도구입니다.

### 2. RESTful API란 무엇일까요?

수많은 API들 중에서, **REST(Representational State Transfer)**라는 특정 '디자인 규칙(아키텍처 스타일)'을 아주 잘 지켜서 만든 API를 **'RESTful API'**라고 부릅니다. "이 API는 REST 원칙을 충실하게 따랐다"는 의미로 '-ful'을 붙여 표현하는 것입니다.

2000년에 로이 필딩(Roy Fielding)이라는 분이 "웹의 장점을 최대한 활용하려면 어떻게 API를 만들어야 할까?"를 고민하며 제시한 원칙들의 모음이 바로 REST입니다.

#### REST의 핵심 구성 요소

RESTful API는 크게 3가지 요소로 구성됩니다.

1.  **자원 (Resource)**: API로 다루는 모든 정보. 예를 들어 '회원 정보', '게시글', '상품' 등이 모두 자원입니다. 이 자원들은 각각 고유한 주소(URI)를 가집니다.
    *   예: `/users` (모든 회원 정보), `/users/123` (123번 ID를 가진 회원 정보)

2.  **행위 (Verb)**: 자원에 대해 무엇을 할 것인지를 나타냅니다. HTTP라는 통신 규약에 정의된 Method(동사)를 사용합니다.
    *   **GET**: 자원을 **조회** (Read)
    *   **POST**: 자원을 **생성** (Create)
    *   **PUT**: 자원을 **전체 수정** (Update)
    *   **DELETE**: 자원을 **삭제** (Delete)

3.  **표현 (Representation)**: 자원을 어떤 형태로 주고받을지를 의미합니다. 보통 **JSON**이나 **XML** 형식을 많이 사용하며, 현재는 JSON이 대세입니다.

#### RESTful API 예시

'회원(user)'이라는 자원을 관리하는 API를 만든다고 가정해 보겠습니다.

*   **모든 회원 정보 조회**: `GET /users`
*   **ID가 123인 회원 정보 조회**: `GET /users/123`
*   **새로운 회원 추가**: `POST /users` (추가할 정보는 요청 본문에 담아서)
*   **ID가 123인 회원 정보 수정**: `PUT /users/123` (수정할 정보는 요청 본문에 담아서)
*   **ID가 123인 회원 정보 삭제**: `DELETE /users/123`

보시면 주소(URI)에는 자원(명사)만 표현하고, 하고자 하는 일(동사)은 HTTP Method로 명확하게 표현하는 것이 RESTful API의 가장 큰 특징 중 하나입니다. `/getUsers` 와 같이 주소에 동사를 쓰는 것은 RESTful한 설계가 아닙니다.

#### REST의 주요 특징 (제약 조건)

RESTful API가 되기 위해서는 몇 가지 중요한 원칙을 지켜야 합니다.

1.  **클라이언트-서버 구조 (Client-Server)**: 요청하는 클라이언트와 자원을 관리하는 서버의 역할을 명확히 분리합니다. 이로 인해 서로의 개발에 영향을 주지 않고 독립적으로 발전할 수 있습니다.
2.  **무상태성 (Stateless)**: 서버는 클라이언트의 이전 요청 상태를 저장하지 않습니다. 각 요청은 독립적으로 처리되어야 하며, 필요한 모든 정보는 요청 자체에 포함되어야 합니다. 이는 서버의 부담을 줄여주고 확장성을 높여줍니다.
3.  **캐시 가능 (Cacheable)**: 서버의 응답은 캐시(임시 저장)가 가능해야 합니다. 자주 요청되는 데이터는 클라이언트나 중간 서버에 저장해두고 재사용하여 성능을 높일 수 있습니다.
4.  **균일한 인터페이스 (Uniform Interface)**: 자원을 식별하는 방식(URI), HTTP Method 사용 등 모든 것이 통일된 규칙을 따라야 합니다. 이는 API를 이해하고 사용하기 쉽게 만듭니다.
5.  **계층형 시스템 (Layered System)**: 클라이언트는 API 서버와 직접 통신하는지, 중간의 프록시 서버나 로드 밸런서를 거치는지 알 필요가 없습니다. 서버 구조를 유연하게 변경할 수 있습니다.

---

### 요약

*   **API**는 프로그램들이 서로 소통하기 위한 **'약속' 또는 '중간 다리'**입니다. 다른 프로그램의 기능을 빌려 쓰거나 데이터를 가져올 때 사용합니다.
*   **REST**는 API를 잘 만들기 위한 **'디자인 설계 원칙'** 모음입니다.
*   **RESTful API**는 이 REST 원칙을 충실하게 지켜서 만든 API를 말합니다.
*   **RESTful API의 핵심**은 **자원(Resource, 명사)**을 **주소(URI)**로 표현하고, **행위(Verb, 동사)**를 **HTTP Method(GET, POST, PUT, DELETE 등)**로 표현하여 직관적이고 재사용하기 쉽게 만드는 것입니다.

### 도움이 될 만한 자료

*   **초보자를 위한 설명**:
    *   AWS - API란 무엇인가요?: [https://aws.amazon.com/ko/what-is/api/](https://aws.amazon.com/ko/what-is/api/)
    *   Red Hat - API란?: [https://www.redhat.com/ko/topics/api/what-are-application-programming-interfaces](https://www.redhat.com/ko/topics/api/what-are-application-programming-interfaces)
    *   AWS - RESTful API란 무엇인가요?: https://aws.amazon.com/ko/what-is/restful-api/
*   **RESTful API에 대한 깊이 있는 이해**:
    *   IBM - REST API란 무엇인가요?: [https://www.ibm.com/kr-ko/topics/rest-apis](https://www.ibm.com/kr-ko/topics/rest-apis)
    *   PoiemaWeb - REST API: [https://poiemaweb.com/js-rest-api](https://poiemaweb.com/js-rest-api)
    *   블로그 - REST API 구성/특징 총 정리: [https://gmlwjd9405.github.io/2018/09/21/rest-api.html](https://gmlwjd9405.github.io/2018/09/21/rest-api.html)
