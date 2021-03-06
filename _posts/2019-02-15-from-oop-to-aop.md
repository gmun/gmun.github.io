---
layout: post
title: "OOP에서 AOP"
tags: [AOP, OOP, Design-Pattern, AspectJ]
categories: [Spring, AOP, OOP]
subtitle: "OOP의 기술적인 한계 그리고 AOP의 마법"
feature-img: "md/img/thumbnail/from-oop-to-aop.png"
thumbnail: "md/img/thumbnail/from-oop-to-aop.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# AOP의 마법

---

### 들어가기전

컴퓨터 프로그래밍의 패러다임은 늘 변화한다.

_절차적 프로그래밍 → 객체 지향 프로그래밍(OOP) → 관점 지향 프로그래밍(AOP)_

새로운 패러다임은 이전 패러다임의 한계를 느끼고 이를 개선한 방식을 제시한다. 그렇다면 완벽하기만 했던 OOP가 어떠한 기술적인 한계가 있길래 AOP(Aspect Oriented Programming)가 등장했는지 궁금증이 생겼다. AOP는 흔히 마법과 같다고 말한다. 본 포스팅에선 그 이유에 대한 해답이 되길 바란다.

Spring Boot 2.0 환경에서 학습을 진행하였고 AOP는 AspectJ로 구현하였다. 또한, 본 포스팅에선 OOP의 디자인들을 통해 OOP의 기술적인 한계를 구조적 관점으로 살펴보았기 때문에 자세한 코드들은 생략되었다. 학습 과정에서 사용했던 코드 및 OOP의 디자인과 Bean을 설정하는 자세한 과정은 [GitHub](https://github.com/gmun/fromooptoaop)를 참고하길 바란다.

### 학습목표

1. OOP의 기술적인 한계
2. AOP의 사용성의 이해

### OOP의 모듈의 재사용과 다양한 적용

하나의 프로세스는 하나의 기능이라는 개발 관점이 지배적이던 시절에 OOP의 등장은 혁신과 같았다.

절차적 프로그래밍은 동작의 관점에서 프로그래밍하므로 데이터의 흐름이 굉장히 중요했다. 반면 OOP는 프로그램의 프로세스와는 별개로 데이터 추상화 작업을 통해 동작과 데이터를 객체라는 하나의 모듈로 정의하고 이를 재사용함으로써 프로그래밍을 한다. 자연스레 객체 간의 구조가 중요하고, 흔히 OOP에선 낮은 결합 도와 높은 응집도를 가진 구조가 이상적인 구조라 말한다.

#### 1.1. 관심사의 분리

이러한 이상적인 구조를 설계하기 위해 고안된 여러 디자인 원칙과 패턴이 존재한다. 그중에 [관심사의 분리(Separation of Concerns)](https://en.wikipedia.org/wiki/Separation_of_concerns)라는 디자인 원칙은 프로그래밍에 있어 OOP의 근간을 잘 지키는 원칙이라할 수 있다. 그렇다면 `관심사의 분리`란 무엇일까. 다음 클래스의 구조를 살펴보자.

![img](/md/img/aop/from-oop-to-aop/class-diagram1.png)

``` java
public class ****Business {
    private StopWatch stopWatch = new StopWatch();

    public void doAction() {
        stopWatch.start();

        // ... 비즈니스 로직 생략

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint()); // 시간 측정
    }
}
```

- doAction() → 비즈니스 로직(핵심 관심사) + 실행시간 측정 로직(횡단 관심사)

먼저 비즈니스 클래스들의 doAction() 메소드안을 살펴보면 실행시간을 측정하는 로직과 비즈니스 로직이 공존하고 있다. AOP의 용어에선 비즈니스 로직을 `핵심 관심사`(Core Concerns)라 하고 그 외 메소드 측정 기능과 같은 부가기능을 `횡단 관심사`(Cross-cutting Concern)라 한다. 이처럼 하나의 객체에 두 관심사가 공존하고 있는 경우 **`비즈니스 로직을 파악하기 어렵고`** 또한 **`부가기능의 관리가 쉽지 않다.`**

#### 1.2. Target과 Aspect

따라서 관심사의 기준으로 `핵심 관심사 모듈`(Target)과 `횡단 관심사 모듈`(Aspect)로 분리하여 각 관심사에 맞게 관리해야 한다.

![img](/md/img/aop/from-oop-to-aop/class-diagram2.png)

- `****Business.class` → 비즈니스 로직의 모듈화
- `SimplePerformanceMonitor.class` → 실행 시간을 측정하는 로직의 모듈화

이처럼 관심사들을 독립적인 모듈로써 관리만 할 수 있다면야 기존에 제기되었던 문제들을 해결할 수 있을 것이다.

### OOP와 디자인 패턴

하지만 여기서 **＂**`어떻게 Target에 Aspect를 적용할 수 있을까?`**”** 라는 해결해야 할 문제가 생긴다. 이 문제에 대해선 일반적으로 사용되고 있는 여러 디자인 패턴을 활용하여 구조적으로 해결하고 부가기능이 독립적인 모듈로써 제 기능을 할 수 있는지 같이 살펴보자.

1. Aspect가 적용되는지
2. Aspect가 탈부착/관리가 쉬운지
3. Aspect가 독립적인 모듈인지

#### 2.1. 상속의 템플릿 메소드 패턴

첫 번째 생각할 수 있는 해결 방안은 상속이다.

상속은 OOP에서 기능을 확장하기 위한 가장 보편적인 접근 방법이 아닐까 생각한다. 이러한 상속의 특징을 활용하여 고안된 디자인 패턴이 바로 템플릿 메소드 패턴이다. GoF Design Pattern 책의 내용을 인용하자면 템플릿 메소드 패턴은 뼈대를 구축하기 위한 패턴이라 정의되어 있다.

>Defines the skeleton of an algorithm in a method, deferring some steps to subclasses. Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithms structure. - GoF Design Pattern

따라서 템플릿 메소드 패턴은 상위 클래스에서 전체적인 흐름을 잡아주고 상속받은 하위 클래스에선 상세한 로직을 오버라이딩하여 구현하는 방식으로 개발된다.

<img src="/md/img/aop/from-oop-to-aop/template-method1.png" style="max-height:none;">

1. `****Business` 클래스들의 공통된 `doAction()` 메소드는 추상화 메소드로 정의하여 일반화한다.
2. 부가기능의 적용 여부를 하위 클래스에서 정할 수 있도록 상위 클래스에 Hooking 목적을 띈 `isMonitoring()` 메소드를 정의한다.
3. `doActionWithMonitoring()` 메소드에서 메소드 실행시간 측정 부가기능을 적용한다.
4. `****Business` 클래스는 상위 클래스에서 `doAction()` 메소드를 재정의하여 비즈니스 로직을 구현한다.

``` java
abstract public class SimplePerformanceMonitor {
    private final StopWatch stopWatch = new StopWatch();

    // 하위 클래스에게 기능 구현을 맡김
    abstract protected void doAction();

    // Hooking Method 실행시간 측정 여부 목적
    protected boolean isMonitoring() {
      return false;
    }

    // 부가기능을 접목시킨 메소드
    final protected void doActionWithMonitoring() {
        if(isMonitoring()) {
            stopWatch.start();
        }

        this.doAction(); // 비즈니스 로직

        if(isMonitoring()) {
            stopWatch.stop();
            System.out.format("time : %s ms\n", stopWatch.getLastTaskTimeMillis());
        }
    }
}
```

#### 구현 이슈

``` html
담당자 비즈니스 로직 수행중... time : 1008 ms    ← Hooking 메소드 활성화
사원 비즈니스 로직 수행중...                     ← Hooking 메소드 비활성화
```
~~1. Aspect가 적용되는지~~~<br/>

테스트 결과를 보면 첫 번째 이슈는 해결되었다. 하지만 하나의 상위 클래스에서 여러 부가기능을 해결하려 했기 때문에 Aspect를 추가하면 할 수록 코드가 복잡해질 수 밖에 없고 Aspect의 탈 부착이 쉽지 않다. 또한, 이처럼 정립된 클래스들을 역으로 일반화해야 하므로 오히려 기존 구조가 복잡해질 수 있다.

**`2. Aspect가 탈부착/관리가 쉬운지`** 에 대한 부분은 `isMonitoring()` Hooking 메소드를 통해 부가기능의 활성화 여부를 하위 클래스에서 정할 순 있다지만 적용해야 할 Target들에 대한 정확한 구조와 공통점들을 파악하기 어렵다면 구현하기 힘들다.

마지막으로 기존 비즈니스 클래스의 코드를 변경해줘야 하고 또한, 상위 클래스의 `doActionWithMonitoring()` 메소드만 봐도 Adivce와 비즈니스 로직이 공존함으로 **`3. Aspect가 독립적인 모듈인지`**  에 대한 부분도 부족하다.

일반적으로 상속을 사용한 디자인 패턴은 유연하지 않고 정적인 구조로 되어 있기에 이러한 문제들이 제기될 수밖에 없다. 특히 Aspect는 수정하거나 추가로 또 다른 Aspect를 부착시키는 작업들이 빈번하게 발생할 수밖에 없다. 따라서 본 패턴을 통해 해결할 수 없고 Aspect를 구조적으로 유연하게 관리할 수 있는 디자인 패턴으로 해결해야 한다.

### 2.2. 프록시를 기반으로 한 패턴들

Aspect와 같은 부가기능을 동적으로 유연하게 관리하기 위해선 Proxy 기반의 디자인 패턴을 사용하면 구조적으로 해결할 수 있다.

Proxy란 대리인 혹은 대리자의 사전적 의미가 있다. 이를 바탕으로 구조적 관점에선 Target을 대신하여 수행되는 객체들을 의미하고 있다. 일반적으로 Proxy는 Target에 접근하는 방법을 제어하거나 Target의 부가적인 기능을 부여하기 위해 사용한다. 디자인 패턴에선 Proxy의 사용되는 목적에 따라 데코레이터 패턴과 프록시 패턴으로 구분된다.

- 데코레이터 패턴 : 부가기능의 목적
- 프록시 패턴 : 접근제어의 목적

Proxy를 이용한 패턴들은 일반적으로 인터페이스를 활용하여 구조적으로 해결하는데 먼저 데코레이터 패턴을 살펴보자.

#### 2.2.1. 데코레이터 패턴

데코레이터 패턴은 Aspect와 같은 부가기능을 동적으로 유연하게 관리할 수 있도록 고안된 Proxy 기반의 디자인 패턴이다.

이 패턴은 인터페이스를 이용한 디자인 패턴으로써 객체의 책임을 전가를 하는 방식으로 기능들을 덭붙여 객체를 완성시키는 데 대표적으로 `java.io` 패키지의 InputStream.class, OutputStream.class가 데코레이터 패턴을 적용한 클래스들이다. 이해를 돕기 위해 생활속에서 예를 들자면 하나의 자동차를 생성한다고 가정해보자.

![img](/md/img/aop/from-oop-to-aop/car-factory-process.png)

독립적으로 각 공정은 맡은 부분만 작업하고 해당 공정의 관심사에 벗어난 작업들은 다음 공정으로 책임을 전가하여 자동차를 완성하는 방식을 따른다. 이와 마찬가지로 데코레이터 패턴도 Aspect를 유연하게 관리하기 위해 각 모듈에서 독립적으로 기능을 덧붙이고 모듈을 점차 발전시키는 구조로 설계된다.

<img src="/md/img/aop/from-oop-to-aop/decorator1.png" style="max-height:none;">

1. 기존 비즈니스 모듈을 인터페이스로 정의한다.
2. `SimplePerformanceMonitor`, `AdminBusiness`는 `Business` 인터페이스를 상속받아 구현한다.
3. 각 데코레이터의 기능의 위임은 생성자나 수정자 메소드를 통해 전달한다.

다음 구조처럼 각 모듈은 자신의 기능을 완성하고 다음 데코레이터 객체로 전달하기 위해 정의한 Business 인터페이스를 생성자나 수정자 메소드를 통해 전달된다. 따라서 데코레이터 패턴은 런타임 시에 다음 위임 대상을 주입받을 수 있도록 위임  순서를 정해줘야한다.

``` java
public Business createAdminBusiness() {
    Business business = new AdminBusiness(); // Target
    business = new SimplePerformanceMonitor(business); // 부가기능 : 실행시간 측정
    return business;
}
```

`createAdminBusiness()` 메소드를 통해 데코레이터의 순서를 정해주고 생성된 `Business` 객체의 기능 테스트를 진행해보면 다음과 같은 결과를 얻을 수 있었다.

``` java
@Test
public void isWeaving() {
    Business amdin = createAdminBusiness();
    admin.doAction();
}
```
``` html
SimplePerformanceMonitor.class ← 부가기능 로직 수행
AdminBusiness.class            ← 비즈니스 로직 수행
담당자 비즈니스 로직 수행중... time : 1001 ms
```

**~~1. Aspect가 적용되는지~~**

다음 결과를 통해 실행시간 측정 로직이 포함된 결과를 얻을 수 있었다. 여기서 중요한 점은 비즈니스의 프로세스이다. 클라이언트의 관점에선 단순히 비즈니스 클래스의 기능이 호출되는 것처럼 보이지만 사실상 SimplePerformanceMonitor에 의해 AdminBusiness 클래스를 요청된다는 걸 알 수 있다.

_Call → 1) SimplePerformanceMonitor(**Proxy**) → 2) AdminBusiness(**Target**) → End_

이처럼 SimplePerformanceMonitor 클래스는 마치 자신이 AdminBusiness 클래스인 마냥 먼저 호출되는 데 이러한 클래스들을 **`Proxy`** 라 한다.

**`2. Aspect가 탈부착/관리가 쉬운지`** 에 대한 부분은 간단하게 Proxy를 추가하여 검증해보면 된다. 예를들어 MemberBusiness가 개발 단계에 있다고 가정하여 MemberBusiness 기능을 호출 시에 강제 에러를 발생시킨다고 가정해보자.

``` java
public Business createMemberBusiness() {
    Business business = new MemberBusiness(); // Target
    business = new SimplePerformanceMonitor(business); // 부가기능 : 실행시간 측정
    business = new AccessProxy(business); // 접근제어 Proxy
    return business;
}

@Test
public void isWeaving() {
    try{
        admin.doAction();
        member.doAction(); // 서비스 불가 Aspect 추가
    }catch(Exception e){
        System.out.println(e);
    }
}
```
``` html
담당자 비즈니스 로직 수행중... time : 1001 ms        ← AdminBusiness
java.lang.RuntimeException: 서비스 준비중입니다.    ← MmemberBusiness
```

**~~2. Aspect가 탈부착/관리가 쉬운지~~**<br/>
**~~3. Aspect가 독립적인 모듈인지~~**

테스트 결과를 보면 기존의 `SimplePerformanceMonitor`, `MemberBusiness` 클래스들의 코드를 수정하는 작업 없이 접근을 제한하는 목적을 띈 `AccessProxy` Aspect 클래스를 추가했다. 결과적으로 부가기능은 각 Proxy에서 독립적으로 관리할 수 있고, 추후 Aspect가 추가되더라도 다음 대상 위임의 순서만 정해주면 최종적인 부가기능이 부착된 비즈니스 객체를 얻을 수 있다는 점을 확인할 수 있었다.

#### 2.2.2. 프록시 패턴

마지막으로 `AccessProxy` 클래스에서 프록시 패턴의 개념을 찾아볼 수 있다. 프록시 패턴은 데코레이터 패턴과 구현 방식은 같지만, Proxy를 사용하는 목적이 다른 디자인 패턴이다.

<img src="/md/img/aop/from-oop-to-aop/proxy1.png" style="max-height:none;">

다음 그림처럼 데코레이터 패턴은 부가기능을 추가를 위해 Proxy를 사용하지만 프록시 패턴은 Target의 라이프 사이클을 자체적으로 관리하기 위해 Target의 직접적인 접근을 제한하는 목적으로 Proxy를 사용한다. 결과적으로 Proxy를 기반으로 하는 디자인을 활용하여 Aspect와 Target을 독립적인 모듈로써 관리할 수 있었고 더 나아가 다음과 같은 이점들을 생각할 수 있었다.

1. 다른 객체에 영향을 주지 않고 동적으로 독립적인 Aspect를 추가할 수 있다.
2. Proxy들은 각 기능에만 집중하면 된다.
3. 각각의 Proxy는 목적이 뚜렷하기 때문에 테스트가 수월하다.
4. 적용할 Aspect가 많아져도 다음 대상 위임의 순서만 정의해주면 유연하게 관리할 수 있다.
5. 언제든 Aspect의 철회가 가능하다.
6. 각 비즈니스 클래스에 필요에 따라 Aspect를 붙이거나 각각 다르게 적용할 수 있다.

#### 구현 이슈

- 구현해야할 과정들이 번거롭다.
- 대상 위임의 은닉성
- 부가기능의 중복

하지만 데코레이터 패턴은 인터페이스를 통해 위임하는 방식이기 때문에 코드 레벨에선 위임의 순서를 미리 알 수 없다. 이러한 익명성으로 중복된 부가기능을 생성할 경우가 발생할 수 있다는 단점이 있다. 또한, Proxy를 기반으로하는 디자인은 기본적으로 인터페이스를 활용하기 때문에 기존 비즈니스 클래스를 인터페이스로 구성하고 각 데코레이터 클래스들의 대상 위임을 설정하는 과정은 번거로울 수밖에 없다.

![img](/md/img/aop/from-oop-to-aop/proxy2.png)

무엇보다 Proxy를 기반으로 하는 디자인들은 인터페이스를 기반으로 구조를 잡기 때문에 인터페이스 자체에 메소드를 추가/수정/제거하는 작업이 동반되면 동시에 구현체마다 각각의 작업을 진행해야 한다는 최악의 단점이 생긴다. 이러한 이유엔 근본적으로 OOP의 특징상 객체와 객체 간의 관계가 필연적으로 생길 수밖에 없기 때문이다.

### 비즈니스에 AOP 적용

그렇다면 구조적으로 어떻게 해결을 해야 할까? 바로 AOP를 활용하면 쉽게 원하는 Target에 부가기능을 적용할 수 있다.

![img](/md/img/aop/from-oop-to-aop/aspectj1.png)

다음 구조를 보면 AOP는 OOP의 구조적인 관계에 대해 얽매이지 않는다는 것을 알 수 있다. 즉 부가기능을 원하는 시점에 적용할 수 있고 비로써 Aspect를 독립적인 모듈로써 온전히 관리할 수 있다는 점이다.

AOP 구현 방식에 대해 자세히 살펴보기 위해선 몇 가지 기본적인 AOP의 용어에 대해 알아야한다.

- Advice : 부가기능의 코드
- JoinPoint : Advice를 적용할 수 있는 여러 시점
- PointCut : 여러 JoinPoint 중에서 최종적으로 Advice를 적용할 JoinPoint
- Weaving : JoinPoint에 Advice하여 핵심기능과 횡단기능이 교차하여 새롭게 생성된 객체를 프로세스에 적용하는 일련의 모든 과정
> 참고 [AOP : Aspect Oriented Programming 개념](https://gmoon92.github.io/spring/aop/2019/01/15/aspect-oriented-programming-concept.html)

<img src="/md/img/aop/from-oop-to-aop/aspectj2.png" style="max-height:none;">

먼저 `****Business.doAction()`에 Adivce를 적용시키기 위해 PointCut를 정의해준다.

``` java
pointcut businessCalls()
:call(* com.gmun.fromooptoaop.design.aspectj..*Business.doAction(..));
```

Target의 호출 시점을 제어할 수 있는 Around Advice를 사용하고 Target의 메소드 호출은 `proceed();`를 통해 호출 시점을 고려하여 부가기능을 구현해준다. 테스트 코드를 통해 Weaving이 적용됐는지 검증해보자.

``` java
@RunWith(SpringRunner.class)
@SpringBootTest
public class AspectJTest {

    @Autowired
    private AdminBusiness admin;

    @Autowired
    private MemberBusiness member;

    @Test
    public void isWeaving(){
        admin.doAction();
        member.doAction();
    }
}
```
``` html
AdminBusiness.doAction()
aspectJ 적용 테스트  com.gmun.fromooptoaop.design.aspectj.AdminBusiness@167279d1 > time : 1000 ms
MemberBusiness.doAction()
aspectJ 적용 테스트  com.gmun.fromooptoaop.design.aspectj.MemberBusiness@138caeca > time : 501 ms
```

~~1. Aspect가 적용되는지~~<br/>
~~2. Aspect가 탈부착/관리가 쉬운지~~<br/>
~~3. Aspect가 독립적인 모듈인지~~<br/>

### 마무리

OOP의 한계에 대해 알아보았다. 무엇보다 관심사의 분리 어려움에 대한 OOP의 근본적인 문제는 객체 간의 관계에 있다는 점이다. 따라서 AspectJ를 통해 AOP를 적용하여 이러한 관계의 문제점에 대해 해결해보았다.

하지만 Spring AOP는 AspectJ가 아닌 기존 Proxy를 기반으로 관계에 대한 문제를 해결하고 있다. 이러한 의문은 다음 포스팅에서 작성해보도록 하겠다.
