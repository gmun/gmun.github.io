---
layout: post
title: "AOP : Aspect Oriented Programming 개념"
tags: [AOP]
categories: [Spring, AOP]
subtitle: "Spring AOP를 들어가기전 AOP 용어와 개념"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
display: "false"
changefreq: daily
priority: 1.0
---

<!--more-->

# AOP의 이해와 원리

---

### 들어가기전

  본 포스팅에선 AOP의 학습에 앞서 기초적인 개념에 대해 상세히 다룰 예정이다. 따라서 AOP가 무엇인지 AOP와 관련된 용어에 대한 설명과 기존 자바에서 AOP의 구현 방식에 대해 작성할 예정이다.

### 학습목표

1. AOP 등장배경
2. AOP 개념과 용어
3. 기존 자바에서 AOP 구현 방식

### AOP이란

AOP는 컴퓨터 패러다임의 일종으로 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍이라 불린다.

일반적으로 Asepect를 관점으로 해석하지만, 사전적 의미를 찾아보면 또 다른 의미를 발견할 수 있다.

>- 문법 형식의 하나로써 반복(反復) 등을 나타내는 동사의 형태.
>   - 반복을 나타내는 동사 → 반복된 코드

다음과 같은 의미를 빗대어 프로그램 측면에서 해석하자면 반복되는 코드가 이에 해당한다고 볼 수 있다. 이러한 코드는 일반적으로 비즈니스 로직과 관계없는 중복된 코드일 가능성이 크고, 이 코드들을 횡단 관심사(Crosscutting Concerns)라 표현한다.

#### 관심 분리(Separation of Concerns)

 AOP에선 이러한 관심사(Concerns)를 기준으로 크게 핵심 관심사와 횡단 관심사로 분류하고 있다.

- 핵심 관심사(Core Concerms)
- 횡단 관심사(Crosscutting Concerns)

핵심 관심사는 프로그램의 핵심 가치와 목적이 그대로 드러난 관심 영역을 뜻한다.  해당 프로그램의 비즈니스 로직이 그러하다.

반면 횡단 관심사는 비즈니스 로직과는 다른 관심 영역을 뜻한다. 프로그램 관점에서의 대표적인 횡단 관심사는 다음과 같다.

- Security
- Profiling
- Logging
- Transaction Management

이처럼 보안, 프로파일링, 로그, 트랜잭션은 핵심 로직은 아니지만, 요구 상황에 따라서 다수의 핵심 로직에 포함할 수 있는 공통 로직들이다.

정리하자면 횡단 관심사란 비즈니스 로직과는 별개의 영역으로 대다수 기능에서 발생하는 중복된 공통 로직을 의미한다.

 또 다른 측면으로 생각해보자면 OOP에서도 추상화와 템플릿 메소드 패턴과 같은 디자인 패턴을 통해 횡단 관심사를 분리할 수 있다. 그렇다면 굳이 OOP가 아닌 AOP라는 새로운 프로그래밍이 등장했을까?

### 등장배경

 일반적으로 새로운 컴퓨터 프로그래밍들이 제시되는 근본적인 이유는 기존의 프로그래밍 단점을 보완하는 데에 있다. 절차적 프로그래밍을 보완하고자 OOP가 등장했던 것처럼 말이다.

_절차적 프로그래밍 → 객체 지향 프로그래밍(OOP) → 관점 지향 프로그래밍(AOP)_

본론으로 돌아와서, AOP는 OOP를 보완하고자 등장한 패러다임이다.

이 점은 매우 흥미로웠다. OOP는 상속과 추상화, 동시에 다양한 디자인 패턴을 통해 기능의 분리와 유연한 확장을 구현할 수 있다. 이를 통해 횡단 관심사 또한 분리할 수 있다. 이처럼 완벽하게만 보였던 OOP에 도대체 어떤 한계가 있었기에 AOP라는 새로운 패러다임이 등장했었을까?

#### OOP의 극단적인 추상화의 한계

 OOP(Object oriented Programming)는 객체와 클래스에 초점을 맞춘 프로그래밍 기법이다. 이러한 OOP의 가장 큰 장점은 추상화를 통해 기능의 분리를 하여 유연한 기능의 확장을 할 수 있다는 점이다.

따라서 횡단 관심사와 핵심 관심사를 하나의 로직에서 분리하고 각각의 독립적인 모듈로 관리하기 위해선 먼저 일반화(Generalization)를 통해 분리할 수 있다.

그다음 유연한 확장을 하기 위해 원래 클래스는 추상화로 구현하고 추상화 클래스에 횡단 관심사가 적용해 이를 구현 객체를 사용한다.

여기까지 OOP를 활용하여 관심사를 분리와 적용을 완벽하게 구현하였다. 하지만 다음과 같은 상황이 닥친다면 어떻게 될까?

 1. 특정 메소드만 적용
 2. 다른 횡단 관심사를 추가로 적용

이에 대응하기 위해선 이에 맞는 각기 다른 추상화 클래스가 필요하다. 결과적으로 추상화 클래스는 유연한 기능의 확장이라는 본질적인 장점과는 다르게 많은 추상화 클래스가 생기게 되고 오히려 이를 관리하는데 큰 비용이 든다.

결과적으로 OOP로 관심사를 분리할 수 있지만, OOP는 객체의 관점으로 횡단 관심사를 분리하기 때문에 앞서 설명과 같이 극단적인 추상화 클래스들이 발생하고, 이를 관리해야 한다는 커다란 숙제가 남게 된다.

#### AOP의 등장 그리고 목표와 방향성

이러한 문제점들을 보안하고자 등장한게 바로 AOP이다.

>[In computing, aspect-oriented programming (AOP) is a programming paradigm that aims to increase modularity by allowing the separation of cross-cutting concerns. ... <br/> ... It does so by adding additional behavior to existing code (an advice) without modifying the code itself  ... - Wikipedia AOP](https://en.wikipedia.org/wiki/Aspect-oriented_programming)

Wikipedia에 정의된 글을 보면 AOP는 "횡단 관심사의 분리를 허용함으로써 모듈성을 증가"라는 목표를 두고 있다.  따라서 AOP는 핵심 관심사와 횡단 관심사를 분리하여 관리함으로써 비즈니스 로직의 모듈성을 증가할 수 있다.

_횡단 관심사와 핵심 관심사를 분리 → 모듈성 증가_

앞서 OOP는 객체를 통해 횡단 관심사의 분리를 허용했다면 AOP는 Aspect를 통해 횡단 관심사를 분리한다. Aspect는 여러 객체를 분리하고 적용하는 동작을 한다.

 결과적으로 AOP의 가장 큰 핵심은 Aspect를 통해 비즈니스 로직에 별도의 코드 추가 없이 횡단 관심사를 분리하거나 동시에 하나의 동작을 여러 객체에 교차로 적용할 수 있다.

이를 통해 Aspect가 동작을 어떻게 하는지 AOP가 준비된 개발자라면 공통 모듈을 개발할 때 수월하게 개발을 할 수 있게 된다.

하지만 AOP를 학습하는 데 많은 어려움이 있다. 가장 근본적인 이유는 생소한 용어들이다.

### 난해한 AOP의 개념과 용어

AOP의 용어는 AOP가 어떻게 동작하는지 그림과 함께 이해하면 많은 도움이 된다.

- Aspect : 여러 객체를 가로 지르는 문제의 모듈화. 트랜잭션 관리는 J2EE 애플리케이션에서 교차하는 문제의 좋은 예입니다. Spring AOP에서 aspect는 정규 클래스 (스키마 기반 접근법) 또는 @Aspect 주석 ( @AspectJ 스타일)으로주석된 일반 클래스를 사용하여 구현된다.
- Joinpoint : 메소드 실행이나 예외 처리와 같은 프로그램 실행 중 포인트. Spring AOP에서 join point는 항상 메소드 실행을 나타낸다. org.aspectj.lang.JoinPoint 유형의 매개 변수를 선언하여 조인 포인트 정보를 조언 본문에서 사용할 수 있습니다.
- Advice(interceptor method) : 특정 조인 포인트에서 한 측면에 의해 취해진 행동. 다양한 유형의 조언에는 "주변", "이전"및 "후"조언이 포함됩니다. 조언 유형은 아래에 설명되어 있습니다. Spring을 포함한 많은 AOP 프레임 워크는 인터셉터 로서 조언을 모델링하고 , 조인 포인트 주변의 인터셉터 체인을 유지합니다.
- Pointcut : 조인 포인트와 일치하는 술어. 조언은 pointcut 표현식과 관련이 있으며 pointcut과 일치하는 조인 포인트에서 실행됩니다 (예 : 특정 이름의 메소드 실행). Pointcut 표현식과 일치하는 조인 포인트의 개념은 AOP의 핵심입니다 : Spring은 기본적으로 AspectJ pointcut 언어를 사용합니다.
- Introduction(inter-type) : 유형을 대신하여 추가 메소드 또는 필드를 선언합니다. Spring AOP를 사용하면 프록시 된 객체에 새로운 인터페이스 (및 해당 구현)를 도입 할 수 있습니다. 예를 들어 소개를 사용하여 Bean이 캐시를 단순화하기 위해 IsModified 인터페이스를구현하도록 할 수 있습니다.
- Target Object : 하나 이상의 특성에 대해 조언을받는 개체입니다. 권고 된 객체라고도합니다. Spring AOP는 런타임 프록시를 사용하여 구현되기 때문에이 객체는 항상 프록시 객체입니다.
- AOP proxy : 애스펙트 계약을 구현하기 위해 AOP 프레임 워크에 의해 생성 된 객체입니다 (메소드 실행 권고 등). Spring 프레임 워크에서 AOP 프록시는 JDK 동적 프록시 또는 CGLIB 프록시가 될 것이다. 프록시 생성은 Spring 2.0에서 소개 된 aspect 선언의 스키마 기반 및 @AspectJ 스타일의 사용자에게는 투명합니다.
- Weaving : 다른 응용 프로그램 유형 또는 개체와 측면을 연결하여 권고 된 개체를 만듭니다. 이것은 컴파일 타임 (예 : AspectJ 컴파일러 사용),로드 시간 또는 런타임에 수행 할 수 있습니다. Spring AOP는 다른 순수 자바 AOP 프레임 워크와 마찬가지로 런타임에 위빙을 수행한다.

#### Advice

Before advice : 조인 포인트 이전에 실행되지만 실행 흐름이 조인 포인트로 진행하지 못하도록하는 조언 (예외가 발생하지 않는 한).

return advice : join point가 정상적으로 완료된 후 실행될 조언 : 예를 들어 메소드가 예외를 발생시키지 않고 리턴하는 경우.

throwing advice : 예외를 throw하여 메소드가 종료 될 경우 실행될 조언.

After (finally) advice : 조인 포인트가 종료되는 방법에 관계없이 실행될 조언 (정상 또는 예외적 복귀).

around advice : 메소드 호출과 같은 조인 포인트를 둘러싼 조언. 이것은 가장 강력한 조언입니다. around advice는 메소드 호출 전과 후에 사용자 정의 동작을 수행 할 수 있습니다. 또한 조인 포인트로 진행할지 또는 자체 반환 값을 반환하거나 예외를 throw하여 권고 된 메소드 실행을 바로 가기할지 여부를 선택하는 작업도 담당합니다.


- 용어
   - Aspect
   - Join point
   - Advice(interceptor를 )
       - Before
       - After returning
       - After throwing
       - Around
   - pointcut
   - introduction
   - target
   - *AOP proxy
   - Weaving

### 3 기존 자바의 AOP 구현 방식

- JDK Dynamic Proxy
- CGLIB
- AspectJ


   - Runtime(동적)      : Jdk Dynamic Proxy, CGLIB - 프록시 기반
   - Compile time(정적) : AspectJ    - 타깃 기반 (타깃 오브젝트를 직접 조작하는 방식)

   Jdk Dynamic Proxy (InvocationHandler)
       - 타겟 메소드가 호출될 때 Advice를 적용

   CGLIB(MethodInterceptor)
       - 메써드가 처음 호출 되었을때 동적으로 bytecode를 생성하여 이후 호출에서는 재사용
       - 클래스에 대한 Proxy가 가능

   프록시 기반
       기본은 인터페이스의 유무에 따라 나눠짐

       y : jdk Dynamic Proxy
       n : cglib


참고[https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ]


4. 스프링 proxy

https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/
6. 스프링은 어떤 방식으로 AOP를제공하고 있을까 ?

   JDK Dynamic Proxy
   CGLIB



6-1. 스프링 AOP의 구현방식
    - XML
    - @AspectJ
    - 비교 분석 장단점



6-2. 스프링 포인트컷
   - 기본
   - 어노테이션

6-3. 어드바이스 라이프 사이클


6-4. weaving 3가지 방식

### 마무리


---

### 참고

[블로그 - Spring 횡단관심(Crosscutting Concerns), 핵심관심(Core Concerns) ](http://winmargo.tistory.com/89)

[해외 - Aspect-Oriented Programming vs. Object-Oriented Programming](https://study.com/academy/lesson/aspect-oriented-programming-vs-object-oriented-programming.html)

[해외 - the-basics-of-aop](https://blog.jayway.com/2015/09/07/the-basics-of-aop/)

[AOP 슬라이드](https://slideplayer.com/slide/9380068/)

[AOP 정부 프레임워크 DOC](http://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj)

- 전반적인 개념
[블로그 - 스프링 AOP(Aspect Oriented Programming)](http://closer27.github.io/backend/2017/08/03/spring-aop/)
[AOP 구현 세가지 방법 비교](https://www.reimaginer.me/entry/AOP-%EA%B5%AC%ED%98%84-%EC%84%B8%EA%B0%80%EC%A7%80-%EB%B0%A9%EB%B2%95-%EB%B9%84%EA%B5%90%EC%97%90-%EA%B4%80%ED%95%9C-%EC%A7%A7%EC%9D%80-%EA%B8%80-JAVA-proxy-CGLIB-AspectJ)
[AOP의 구조 + 어노테이션](https://hunit.tistory.com/188)
[해외 블로그 - Implementing AOP With Spring Boot and AspectJ](https://dzone.com/articles/implementing-aop-with-spring-boot-and-aspectj)
[블로그 - 스프링이 제공하는 aop](https://minwan1.github.io/2017/10/29/2017-10-29-Spring-AOP-Proxy/)

- aspect
[스프링 DOC - AOP aspect](https://docs.spring.io/spring/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)
[스프링 블로그 - aspectj](https://www.baeldung.com/aspectj)
[스프링 DOC - Spring을 이용한 Aspect 지향 프로그래밍](https://docs.spring.io/spring/docs/2.0.x/reference/aop.html)

- pointcut
[스프링 블로그 - pointcut](https://www.baeldung.com/spring-aop-pointcut-tutorial)
[블로그 - 3. 스프링 AOP (AspectJ의 Pointcut 표현식) ](http://blog.naver.com/PostView.nhn?blogId=chocolleto&logNo=30086024618&categoryNo=29&viewDate=&currentPage=1&listtype=0)

- adivce
[해외 블로그 - @After](https://howtodoinjava.com/spring-aop/aspectj-after-annotation-example/)

- 실제 사용법
[스프링 부트에서 aspectJ 형식으로 코드 참고](http://jsonobject.tistory.com/247)

---

- 동영상

[스터디 스프링5 입문 - AOP 프로그래밍 1 #10](https://www.youtube.com/watch?v=wrHTMsKrKkA&index=6&list=WL&t=0s)
[스터디 스프링5 입문 - AOP 프로그래밍 2 #11](https://www.youtube.com/watch?v=9Gdv6fhhaB0&index=5&list=WL&t=0s)
[스터디 코드로배우는스프링 38 Spring의 AOP](https://www.youtube.com/watch?v=4-JcM7y1M_8&index=7&list=WL&t=0s)
[What is AOP - Aspect Oriented Programming](https://www.youtube.com/watch?v=DuFPj8MlAVo&index=8&list=WL&t=0s)
[신입SW인력을 위한 실전 자바(Java) 스프링(Spring) 동영상과정 제 09강 AOP-I](https://www.youtube.com/watch?v=2F8K9BLgvjE&index=9&list=WL&t=0s)

---
블로그

[Filter, Interceptor, AOP의 흐름](https://doublesprogramming.tistory.com/133)
[Spring Filter, Interceptor AOP 차이 및 정리 ](http://goddaehee.tistory.com/154)
[filter, interceptor, aop의 차이와 그 목적](http://hayunstudy.tistory.com/53)