---
layout: post
title:  "SingleTon Pattern"
date:   2021-11-14 09:01:09 +0900
categories: jekyll update
---

 ```
목적
: Singleton 패턴의 정의와 장점과 단점을 알아보고, 예시를 통하여 언제, 어떻게 사용하는 것이 좋은지 학습
: 자바의 Singleton과 Spring의 Singleton이 어떻게 다른지 학습
```

- Singleton Pattern이란?

[소프트웨어 디자인 패턴](https://ko.wikipedia.org/wiki/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4_%EB%94%94%EC%9E%90%EC%9D%B8_%ED%8C%A8%ED%84%B4)에서 **싱글턴 패턴**(Singleton pattern)을 따르는 클래스는, 생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나이고 최초 생성 이후에 호출된 생성자는 최초의 생성자가 생성한 객체를 리턴한다. 이와 같은 디자인 유형을 **싱글턴 패턴**이라고 한다.   주로 공통된 객체를 여러개 생성해서 사용하는 DBCP(DataBase Connection Pool)와 같은 상황에서 많이 사용된다.

-WIKI (https://ko.wikipedia.org/wiki/%EC%8B%B1%EA%B8%80%ED%84%B4_%ED%8C%A8%ED%84%B4)

*
그러므로,
*

여러 쓰레드가 동시에 해당 인스턴스를 공유하여 사용할 수 있도록 하기 때문에, 공통으로 사용해야 처리해야 하는 인스턴스를 동시에 사용하는 환경에서 쓰기 적당함


- 장점

여러 쓰레드가 동시에 해당 인스턴스를 공유하여 사용할 수 있도록 하기 때문에 요청이 많은 곳에서 사용하면 효율을 높일 수 있음

- 단점

Singleton 인스턴스가 너무 많은 일을 하거나 많은 데이터를 공유시킬 경우, 다른 클래스의 인스턴스들 간에 의존성이 높아져 수정이 어려워지고, 유지보수 비용이 높아질 수 있음

Singleton 인스턴스를 너무 많이 생성하면 메모리를 점유하여 성능에 악영향을 줄 수 있음

- 예시

```
public class CarClass{

        //클래스로더가 초기화 하는 시점에 정적 바인딩을 통해 해당 인스턴스를 static 메모리에 등록
    //static 관련 설명(https://coding-factory.tistory.com/524)
        private static CarClass car = new CarClass();

        private CarClass(){}

        //사용 요청이 있을 때, 객체를 반환
        public static CarClass getInstance(){
                return car;
        }
        //사용여부 리턴
        private static boolean isUse = false;

        public static void drive(){
                isUse = true;
                System.out.println("start driving");
        }

        public static void parking(){
                isUse = false;
                System.out.pringln("parking");
        }

        public static boolean isEnableUseCar(){
                return !isUse;
        }
}
```

*

멀티 스레드 환경에서 아래와 같은 요청을 처리 함

```
CarClass car = CarClass.getInstance();

if( car.isEnableUseCar ){
    car.drive();
    car.parking();

}else{
    System.out.println("car is already used. plz wait! ");
}
```

*


- 구현방식

    - Eager Initialization

```
public class Singleton {
    // Eager Initialization

    //객체 생성
    private static Singleton uniqueInstance = new Singleton();
    private Singleton() {}

    public static Singleton getInstance() {
        return uniqueInstance;
    }
}
```

클래스 로더가 초기화 하는 시점에서 정적 바인딩(static) 해당 인스턴스를 메모리에 등록하여 사용 함. 클래스가 최초로 로딩 될 때 객체가 생성되기 때문에, Thread-safe 함

Eager Initialization는 무조건 메모리에 올리기 때문에, 필요할 때만 메모리에 올리는 방식으로 Lazy Initialization with sysnchronized 고려할 수 있음

    - Lazy Initialization with sysnchronized

```
public class Singleton {

    private static Singleton uniqueInstance;

    private Singleton() {}

    // Lazy Initailization

    public static synchronzied Singleton getInstance() {
        if(uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }

        return uniqueInstance;
    }
}
```

syschronized 키워드를 이용한 게으른 초기화 방식으로 메서드에 동기화 블럭을 지정하 Tread-safe를 보장함.

단점은 인스턴스를 사용하려면, 무조건 동기화 블록을 거쳐야 하기 때문에, 성능이 떨어진다. 이 단점을 개선한 방식이 Lazy Initialization. Double Checking Locking(DCL, Thread-safe)

    - Lazy Initialization. Double Checking Locking(DCL, Thread-safe)

```
public class Singleton {
    private volatile static Singleton uniqueInstance;

    private Sigleton() {}

    // Lazy Initialization. DCL
    public Singleton getInstance() {

        //객체가 존재하지 않을 때만 synchronized!
        if(uniqueInstance == null) {
            synchronized(Singleton.class) {
                if(uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

이 방법은 객체가 생성되지 않았을 때만 sysnchronized 하기 때문에, 최초 객체가 생성될 시점만 대기가 발생한다.

volatile 키워드를 사용하면 cpu cache가 아니라 main memory에서 값을 읽어오기 때문에, 변수값 불일치 문제가 생기지 않는다.

*
*


## Lazy Initialization. LazyHolder

*
*

```
public class Singleton {

    private Singleton() {}

    /**
     * static member class
     * 내부클래스에서 static변수를 선언해야하는 경우 static 내부 클래스를 선언해야만 한다.
     * static 멤버, 특히 static 메서드에서 사용될 목적으로 선언
     */
    private static class InnerInstanceClazz() {
        // 클래스 로딩 시점에서 생성
        private static final Singleton uniqueInstance = new Singleton();
    }

    public static Singleton getInstance() {
        return InnerInstanceClazz.instance;
    }
}
```

*
*

LazyHolder 방식은 가장 많이 사용하는 singleton 구현 방식으로 volatile이나, synchronized 키워드 없이도 동시성 문제를 해결하기 때문에, 성능이 뛰어남

원리는 InnerInstanceClazz 클래스에 변수가 없기 때문에, static이라도 클래스 로더 초기화 시, InnerInstanceClazz 를 메모리에 올리지 않고, getInstance() 메서드를 호출 시, 초기화 됨.

InnerInstanceClazz 내부 인스턴스는 static 이기 때문에 클래스 로딩 시점에 한번만 호출된다는 점을 이용한것이며, final을 써서 다시 값이 할당되지 않도록 함

- Singleton 구현 시, 주의사항

    - 클래스 로더를 2개 이상 사용하는 경우, 인스턴스가 2개 이상 생성될 수 있기 때문에, 클래스 로더를 지정해야 함

        - java와 spring Singleton 차이점
            - java의 경우, 톰캣이 war파일을 만들면 war파일 하나당 클래스 로더 하나 1:1 식으로 배치가 됨. 다른 war 파일은 참조가 불가능
            - Spring의 경우, web.xml 에서 root context 하나와 servlet context 여러개를 등록 할 수 있는데, 이 각각의 context들이 Singleton 범위가 됨

    - 멀티 스레딩 환경에서 동작 가능하도록 구현해야 하기 떄문에, 무상태성을 지켜야 함

```
@RequiredArgsConstructor

/*https://webdevtechblog.com/requiredargsconstructor-%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85-dependency-injection-4f1b0ac33561*/

//빈에 생성자가 하나만 있고, 파라미터 타입이 빈으로 등록 가능한 존재라면, @Autowired 없이 의존성 주입이 가능

public class EventService {
    private final ApplyRepository; // (1)
    private List<Stirng> applicants; // (2)
    private ApplyVo apply; // (2)

    public void createEvent() {
        // 생략
    }
}
```

        - (1) 다른 Singleton 빈을 저장하려는 용도로 사용 가능
        - (2) 여러개의 스레드가 접근하는 경우, 값이 변경될 위험이 있기 때문에 Thread-safe 하지 않음

*
*

- Spring의 Singleton 패턴

    - 스프링은 빈을 등록 시, 범위를 지정할 수 있음
        - prototype : 컨테이너에 빈을 요청할 때마다 매번 새로운 객체를 만듬
        - request : HTTP 요청 하나당 하나의 객체를 만듬

Spring에서 싱글턴을 저장하고 관리해주는 것이 applicationContext (Singleton Registry, IOC 컨테이너, 스프링컨테이너, 빈팩토리)

*
*
Spring 핵심 컨테이너의 빈 관리를 담당하는 빈팩토리의 핵심 구현 클래스는 DefaultListableBeanFactory 임


- 결론

처음에는 Singleton 패턴이 무엇이고 언제 쓰고 어떻게 쓰는지 정도만 살펴보려 했는데 찾아보다 보니, Singleton 패턴이 Spring 프레임워크에 녹아 있는 패턴이라는 것을 새삼 알게 되었다. 이 패턴을 기반하여 내가 프로그래밍 해왔구나 생각하니 너무 모르고 쓰고 있다는 생각이 들었다.

또한 멀티 스레드 환경에서 동시성 제어관련해서도 DB에서 다 하고 있고, spring의 빈팩토리에서 다 해주고 있기 때문에 인지하지 못했는데, 특히 빈팩토리의 싱글톤 범위, 기법에 대해 더 살펴봐야 겠다는 생각이 든다. 왜냐하면 어떻게 사용하느냐에 따라 시스템의 성능을 좌지우지 할 수 있기 때문이다.
