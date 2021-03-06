---
layout: post
title: "AOP가 왜 필요할까?"
tags: [AOP, Spring AOP]
categories: [Spring, AOP]
subtitle: "OOP의 등장과 한계 그리고 AOP가 필요한 이유"
feature-img: "md/img/thumbnail/aop.png"
thumbnail: "md/img/thumbnail/aop.png"
excerpt_separator: <!--more-->
sitemap:
changefreq: daily
priority: 1.0
---

<!--more-->

# OOP의 등장과 한계 그리고 AOP가 필요한 이유

---

### 들어가기전

컴퓨터 프로그래밍의 패러다임은 늘 변화했습니다.

_절차적 프로그래밍 → 객체 지향 프로그래밍(OOP) → 관점 지향 프로그래밍(AOP)_

하지만 개발자들은 새로운 프로그래밍이 왜 등장했고, 왜 필요한지 이해를 하지 않고 사용하는 경우가 대부분입니다. 본 포스팅은 OOP가 등장한 이유와 왜 우리가 현재 사용하고 있는지 그리고 관점지향 프로그래밍이라 불리는 AOP가 왜 필요하고 등장했는지 알기위해 작성되었습니다.

### 학습목표

1. OOP의 등장한 이유
2. AOP에 대한 이해

### 1. OOP의 등장배경

우선, 현재는 많은 프로젝트에서 절차적 프로그래밍 대신 OOP 패러다임을 지향하며 프로그래밍을 하고 있습니다. 이러한 이유엔 프로젝트의 규모가 커질수록 절차적 프로그래밍이 적합하지 않기 때문이라 생각합니다.

절차적 프로그래밍은 데이터보단 프로세스의 동작을 중요하기 때문에, 하나의 프로세스가 잘 동작하기 위해 데이터의 흐름이 굉장히 중요했습니다.

![proedural-flow](/md/img/aop/why-used-aop/procedural-flow.png)

다음 그림처럼 함수에서 함수로 전달할 데이터가 중요했고, 무엇보다 이 함수들은 하나의 업무 프로세스에 구성되어 있어 데이터 흐름이 명확하므로 업무의 파악이 쉽다는 장점이 있습니다.

![spaghetti-code](/md/img/aop/why-used-aop/spaghetti-code.png)

_중복된 함수 발생 → 유지보수 어려움 → 스파게티 코드 발생_

하지만 프로젝트의 규모가 커질수록 데이터 흐름을 맞추기 위한 조건문/반복문의 남용하기 때문에, 명확한 코드를 파악하기가 어렵고 다른 업무 프로세스를 개발 시 중복된 함수가 발생할 가능성이 컸습니다. 또한, 추후 유지 보수하는 과정에서 스파게티 코드가 발생할 가능성이 큽니다. 따라서 정의된 기능들을 재사용하기 위해 동작보다는 객체를 중심으로 프로그래밍하는 OOP가 등장했습니다.

### 1.1. 관심사 분리

OOP의 핵심은 공통된 목적을 띈 데이터와 동작을 묶어 하나의 객체로 정의하기 때문에, 이 객체를 프로그래밍에 적극적으로 활용함으로써 기능을 재사용할 수 있다는 점이 아닐까 생각합니다. 무엇보다 이 객체를 잘 활용하기 위해선 [관심사 분리(separation of concerns, SoC)](https://ko.wikipedia.org/wiki/%EA%B4%80%EC%8B%AC%EC%82%AC_%EB%B6%84%EB%A6%AC)의 디자인 원칙을 준수해야 합니다.

![spring-mvc-soc](/md/img/aop/why-used-aop/spring-mvc-soc.png)

Spring MVC 구조만 봐도 @Controller, @Service, @Repository와 같이 관심사 별로 계층을 나눠 객체를 관리한다든지 또 다른 예로는 다양한 OOP의 디자인 패턴을 봐도 관심사를 분리하기 위한 노력의 결실이라고 볼 수 있습니다.

이처럼 프로그래밍에 있어 이 관심사의 분리는 모듈화의 핵심이고 건강한 애플리케이션의 만드는 필수적인 과정이라 생각합니다.

### 1.2. 관심사의 분리의 한계

아이러니하게도 이 관심사를 분리하는 데 있어, OOP의 한계가 존재합니다.

![separation-of-concerns](/md/img/aop/why-used-aop/separation-of-concerns.png)

다음 그림을 보면 서비스 클래스에 해당 업무 코드에 트랜잭션의 코드가 존재합니다. 이 트랜잭션 코드는 업무(비즈니스)와는 관련이 없지만, 필수적인 부가기능입니다. 이 트랜잭션 기능은 불특정 다수의 클래스에서 존재합니다.

- Cross-cutting Concerns
- Core Concerns

관심사 관점에선 이러한 트랜잭션 코드를 횡단 관심사(Cross-cutting Concerns)라 하고 업무 관련 코드를 핵심 관심사(Core Concerns)라 합니다.

![cross-cutting-concerns](/md/img/aop/why-used-aop/cross-cutting-concerns.png)

- 보안
- 로깅
- 프로파일링
- 트랜잭션

이 횡단 관심사 코드는 트랜잭션을 비롯하여 보안, 로깅, 프로파일링과 같은 공통 기능 성향을 띈 코드들이 프로젝트의 곳곳의 비즈니스 클래스 안에 존재하게 됩니다.

### 1.3. 관심사의 코드의 특징

결과적으로 하나의 비즈니스 클래스에 횡단 관심사와 핵심 관심사가 공존하고 있어, 명확하게 비즈니스 코드를 파악하기가 어렵고 불특정으로 작성된 횡단 관심사 코드들을 관리하기가 쉽지 않습니다.

- 메소드 복잡도 증가 → 비즈니스 코드 파악이 어려움
- 불특정 다수의 메소드에 반복적으로 구현함 → 횡단 관심사의 모듈화가 어려움

따라서 이 횡단 관심사 코드들을 분리해야 하지만 OOP에선 횡돤 관심사 코드를 깔끔하게 분리하고 기존 비즈니스 코드에 적용하기가 매우 어렵습니다.

### 2. AOP의 등장

이러한 OOP의 관심사 분리에 대한 한계적인 부분을 해결하고자 AOP가 등장하게 되었습니다.

![aop-application](/md/img/aop/why-used-aop/aop-application.png)

다음 그림처럼 AOP는 이 횡단 관심사 코드를 모듈화(Aspect)하여 관리합니다. 따라서 더 이상 기존 비즈니스 코드에 횡단 관심사 코드를 작성하지 않아도 됩니다.

### 2.1. 횡단 코드의 분리와 적용

무엇보다 Aspect에 의해 횡단 관심사 코드를 비즈니스 코드에 적용시킬 수 있다는 점은 AOP의 핵심이라 말할 수 있습니다.

![used-aop](/md/img/aop/why-used-aop/used-aop.png)

Aspect 파일에 의해 비즈니스 코드에 횡단 코드의 적용 시점 또는 적용 여부를 쉽게 탈부착하기가 쉽다는 점이 아닐까 생각이 듭니다.

이러한 이유엔 Aspect엔 Pointcut과 Advice라는 코드가 포함되어있습니다. Pointcut은 횡단 관심사 코드를 비즈니스 로직중 어디에 적용을 할지 위치를 나타내고 Advice는 순수한 횡단 관심사 코드를 의미합니다. 따라서 Pointcut과 Advice만 안다면 쉽게 횡단 코드를 원하는 비즈니스 코드에 적용할 수 있습니다.

### 마무리

앞서 AOP의 등장한 이유와 왜 사용해야하는지 간략하게 정리를 해보았습니다.

본론으로 돌아와서 Spring에서도 AOP를 제공하고 있습니다. 그게 바로 Spring AOP입니다. Spring AOP는 AspectJ의 어노테이션 방식으로 AOP를 구현할 수 있도록 구현방식을 제공하고 있습니다. 이러한 AspectJ의 어노테이션의 AOP 구현방식 때문인지는 몰라도 개발자들이 "Spring AOP와 AspectJ는 같은 거 아니었어?" 라는 오해를 하고 있습니다.

Spring AOP와 AspectJ의 차이에 대해 명확히 인지하고 스프링에서 AOP 사용하는 것이 매우 중요합니다.

### 앞으로 다뤄질 내용

- Spring AOP의 동작 방식
- JDK Dynamic Proxy와 CGLib
- Spring AOP와 AspectJ의 차이
