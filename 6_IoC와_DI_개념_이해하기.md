# IoC(제어의 역전)와 DI(의존성 주입) 완벽 이해

Spring 프레임워크의 가장 핵심적인 개념인 `IoC`와 `DI`에 대해 알아보겠습니다. 이 개념을 이해하면 왜 스프링이 강력한 프레임워크인지, 어떻게 우리의 코드를 더 좋게 만드는지 알 수 있습니다.

### IoC와 DI, 왜 필요할까요?

본격적인 설명에 앞서, 왜 우리가 IoC와 DI라는 개념을 배워야 하는지 간단한 비유로 알아볼게요.

당신이 `자동차`를 조립한다고 상상해 보세요.

*   **전통적인 방식:** 당신은 직접 `엔진` 공장에 가서 특정 엔진을 주문하고, `타이어` 가게에 가서 특정 타이어를 사옵니다. 모든 부품을 직접 선택하고 가져와서 조립해야 합니다. 만약 다른 엔진으로 바꾸고 싶다면, 자동차의 설계 자체를 변경해야 할 수도 있습니다.
*   **Spring 방식(IoC/DI):** 당신은 자동차의 `설계도`만 그립니다. 설계도에는 "여기는 엔진이 들어갈 자리", "여기는 타이어가 들어갈 자리"라고 명시만 해둡니다. 그러면 `조립 라인(Spring 컨테이너)`이 알아서 그 자리에 가장 적합한 `엔진`과 `타이어`를 가져와서 "주입"하고 완성차를 만듭니다. 당신은 어떤 회사에서 만든 엔진인지, 타이어 브랜드가 무엇인지 신경 쓸 필요 없이 오직 '설계'라는 핵심 작업에만 집중할 수 있습니다.

이처럼 개발자가 프로그램의 모든 제어권을 갖는 것이 아니라, **Spring이라는 거대한 프레임워크(조립 라인)에게 제어권을 넘겨서** 개발자는 비즈니스 로직(핵심 기능) 개발에만 집중할 수 있도록 하는 것이 바로 IoC와 DI의 핵심 아이디어입니다.

---

### 1. IoC (Inversion of Control, 제어의 역전)

**IoC란 말 그대로 '제어'가 '역전(뒤바뀜)'되었다는 뜻입니다.**

*   **무엇을 제어하는가?** 객체(Object)의 생성, 관리, 연결 등 객체의 생명주기 전반을 의미합니다.
*   **어떻게 역전되었는가?**
    *   **기존 방식:** 개발자가 코드 안에서 `new` 키워드를 사용해 직접 객체를 생성하고 관리했습니다. 객체에 대한 모든 제어권은 개발자에게 있었습니다.
    *   **IoC 방식:** 객체에 대한 제어권이 개발자에게서 **Spring 컨테이너(IoC Container)**로 넘어갑니다. 개발자는 어떤 객체가 필요하다고 Spring에게 알려주기만 하면, Spring 컨테이너가 알아서 객체를 만들고, 관리하고, 필요한 곳에 전달해 줍니다.

**IoC의 장점**
*   **결합도 감소:** 객체들이 서로에 대해 직접적으로 알 필요가 없어집니다. 이로 인해 한 객체의 변경이 다른 객체에 미치는 영향이 줄어들어 유지보수가 쉬워집니다.
*   **유연성과 확장성 증가:** 부품을 갈아 끼우듯이 쉽게 다른 구현체로 변경할 수 있습니다.

---

### 2. DI (Dependency Injection, 의존성 주입)

**DI는 IoC 개념을 실제로 구현하는 핵심적인 방법입니다.** '제어의 역전'이라는 목표를 달성하기 위한 구체적인 기술이라고 할 수 있습니다.

*   **의존성(Dependency)이란?** 하나의 객체가 다른 객체의 기능을 사용해야 할 때, "이 객체는 저 객체에 의존한다"라고 말합니다. 예를 들어, `자동차` 객체는 `엔진` 객체가 있어야 움직일 수 있으므로, `자동차`는 `엔진`에 의존합니다.
*   **주입(Injection)이란?** 말 그대로 '넣어준다'는 뜻입니다. 객체가 필요로 하는 다른 객체(의존성)를 외부(Spring 컨테이너)에서 만들어서 넣어주는 것을 의미합니다.

#### 코드 예제로 이해하기

`Car`가 `Engine`을 사용하는 상황을 코드로 보겠습니다.

**❌ DI가 적용되지 않은 경우 (강한 결합)**

```java
public class Car {
    private V8Engine engine;

    public Car() {
        // Car가 직접 V8Engine 객체를 생성합니다.
        // Car는 V8Engine 클래스에 강하게 의존(결합)하고 있습니다.
        this.engine = new V8Engine();
    }

    public void move() {
        engine.start();
    }
}
```
위 코드의 문제점은 `Car`가 `V8Engine`이라는 특정 클래스를 직접 알아야 한다는 것입니다. 만약 `ElectricEngine`으로 바꾸려면 `Car` 클래스의 코드를 직접 수정해야 합니다.

**✅ DI가 적용된 경우 (느슨한 결합)**

```java
// Car 클래스
public class Car {
    private final Engine engine; // 특정 엔진이 아닌 Engine 인터페이스에 의존

    // 생성자를 통해 외부에서 Engine 구현체를 주입받습니다.
    public Car(Engine engine) {
        this.engine = engine;
    }

    public void move() {
        engine.start();
    }
}

// Engine 인터페이스
public interface Engine {
    void start();
}

// V8 엔진 구현체
public class V8Engine implements Engine {
    public void start() {
        System.out.println("V8 엔진 시동! 부르릉...");
    }
}

// 전기 엔진 구현체
public class ElectricEngine implements Engine {
    public void start() {
        System.out.println("전기 엔진 시동! 윙...");
    }
}
```
이제 `Car`는 `V8Engine`이나 `ElectricEngine` 같은 구체적인 클래스를 몰라도 됩니다. 그저 `Engine`이라는 '역할(인터페이스)'만 알면 됩니다. 실제 어떤 엔진을 사용할지는 외부(Spring 컨테이너)에서 결정하여 `Car`의 생성자를 통해 '주입'해 줍니다. 이렇게 하면 `Car` 코드의 변경 없이도 다양한 엔진을 유연하게 사용할 수 있습니다.

#### Spring의 주요 DI 방법

Spring에서는 의존성을 주입하는 3가지 주요 방법이 있습니다.

1.  **생성자 주입 (Constructor Injection) - ✨가장 권장되는 방법✨**
    *   생성자를 통해 의존성을 주입합니다.
    *   객체가 생성될 때 딱 한 번만 주입되므로 의존성이 누락될 위험이 없고, `final` 키워드를 사용하여 불변성(안전성)을 보장할 수 있습니다.
    *   Spring 공식 문서에서도 가장 권장하는 방식입니다.

    ```java
    @Component
    public class MyService {
        private final MyRepository myRepository;

        @Autowired // Spring 4.3부터 생성자가 하나일 경우 생략 가능
        public MyService(MyRepository myRepository) {
            this.myRepository = myRepository;
        }
    }
    ```

2.  **수정자 주입 (Setter Injection)**
    *   `setter` 메서드를 통해 의존성을 주입합니다. 선택적이거나 변경 가능한 의존성에 사용됩니다.

3.  **필드 주입 (Field Injection)**
    *   변수(필드)에 직접 `@Autowired` 어노테이션을 붙여 주입합니다. 코드가 간결해 보이지만, 외부에서 변경이 불가능하고 테스트가 어려워지는 등 여러 단점이 있어 **현재는 사용을 권장하지 않습니다.**

---

### 요약

*   **IoC (제어의 역전):** 객체를 만들고 관리하는 제어권이 개발자에서 **Spring 컨테이너**로 넘어간 것.
*   **DI (의존성 주입):** IoC를 구현하는 방식으로, 필요한 객체(의존성)를 외부(Spring 컨테이너)에서 만들어서 넣어주는 것.
*   **핵심 장점:** 객체 간의 **결합도를 낮추고(느슨한 연결)**, **유연성과 재사용성을 높여** 유지보수가 쉬운 코드를 만들 수 있습니다.

### 도움이 될 만한 자료

*   **공식 문서:** [Spring Framework Core Technologies - IoC Container](https://docs.spring.io/spring-framework/reference/core.html#spring-core)
*   **추천 블로그:** [김영한님 - 스프링 핵심 원리 이해 2편](https://www.inflearn.com/roadmaps/373) (인프런 로드맵)
