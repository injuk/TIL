# Spring Advanced
## 2024-10-17 Thu
### 스프링의 빌트인 포인트컷 활용하기
* 스프링은 이미 대부분의 경우에 활용 가능한 포인트컷을 제공하고 있으며, 그 예로 아래와 같은 `NameMatchMethodPointcut`이 있다.
```kotlin
class AdvisorTest {
  @Test
  fun `스프링이_제공하는_빌트인_포인트컷`() {
    // given
    val target: ServiceInterface = ServiceImpl()

    // when
    val proxy = ProxyFactory(target).run {
      val pointcut = NameMatchMethodPointcut()
      pointcut.setMappedNames("save") // 부가 기능을 적용할 메소드 이름을 포인트컷에 제공한다.

      val advisor = DefaultPointcutAdvisor(pointcut, TimeAdvice())
      addAdvisor(advisor)

      proxy as ServiceInterface
    }

    // then
    proxy.save()
    proxy.find()
  }
}
```

## 2024-10-18 Fri
### 스프링의 빌트인 포인트컷
* 이렇듯 **스프링은 유용한 포인트컷을 이미 제공하고 있으며, 상술한 포인트컷 외에도 다음과 같은 여러 빌트인 포인트컷을 제공**한다.
  1. `NameMatchMethodPointcut`: 상술한 예시의 포인트컷으로, 내부적으로 `PatternMatchUtils`를 활용하여 메소드 이름을 기반으로 결과를 판단한다.
  2. `JdkRegexpMethodPointcut`: JDK 정규 표현식을 기반으로 포인트컷 결과를 판단한다.
  3. `TruePointcut`: 항상 `true`를 반환하는 방식으로 동작한다.
  4. `AnnotationMatchingPointcut`: 어노테이션을 기반으로 결과를 판단한다.
  5. `AspectJExpressionPointcut`: `aspectJ` 표현식을 기반으로 결과를 판단한다.
* **실무를 기준으로 이 중 가장 중요한 것은 `AspectJExpressionPointcut`으로, 사실상 다른 포인트컷을 크게 중요하지 않다고 이해해도 무방**하다.
  * 이는 **해당 포인트컷의 사용성이 좋을 뿐만 아니라 기능도 가장 많은 `aspectJ` 표현식을 기반으로 동작하기 때문**이다.

## 2024-10-19 Sat
### 프록시에 여러 어드바이저를 함께 적용하기
* 어드바이저는 하나의 포인트컷과 어드바이스 쌍을 갖지만, 용도에 따라서는 여러 어드바이저를 하나의 서버 인스턴스에 대해 적용하고 싶을 수 있다.
  * 언뜻 생각했을 때는 여러 프록시를 활용하는 프록시 체인을 이용하여 클라이언트로부터의 요청이 여러 프록시를 거쳐 서버로 도달하게 할 수 있을 것 같다.
  * 이러한 방식이 불가능한 것은 아니지만 원하는 어드바이스의 개수만큼 프록시를 생성한다는 한계가 수반되며, 이는 고스란히 유지보수 비용 증가로 이어진다.
* **스프링은 이러한 문제를 해결하기 위해 하나의 프록시에 필요한 수만큼 `addAdvisor()` 메소드를 호출하여 어드바이저를 등록할 수 있도록 지원**한다.
  * 이 때, **`addAdvisor()` 메소드를 호출하여 어드바이저를 등록한 순서대로 어드바이저가 호출되어 동작**하게 된다.

## 2024-10-20 Sun
### AOP 및 프록시와 관련된 오해
* 스프링 AOP를 대강 이해할 경우, AOP를 적용한 개수만큼 프록시 인스턴스가 생성될 것으로 착각하기 쉽다.
* 그러나 **스프링은 AOP를 적용하는 과정에서 내부적인 최적화를 진행하므로, 실제로는 프록시 인스턴스를 하나만 생성하고 여러 개의 어드바이저를 적용**한다.
  * 즉, **단일 서버 인스턴스에 여러 AOP를 동시에 적용했더라도 스프링 AOP는 대상 서버 인스턴스마다 단 하나의 프록시 인스턴스만을 생성**한다.

## 2024-10-21 Mon
### 프록시 팩토리의 한계와 빈 후처리기
* 프록시 팩토리를 활용하는 것으로 개발자는 매우 편리하게 프록시를 생성할 수 있으며, 어떤 부가 기능이 언제 어디에 적용될지 명확히 이해할 수 있게 된다.
  * 이렇듯 부가 기능 그 자체와 명확한 적용 시점을 이해할 수 있는 것은 어드바이저와 어드바이스, 포인트컷 개념 덕분이다.
* 프록시 팩토리를 활용할 경우 원본 코드를 전혀 수정하지 않으면서도 프록시의 많은 이점을 누릴 수 있으나, 여전히 다음과 같은 문제점이 남아 있다.
  1. 프록시 팩토리를 적용하기 위한 서버 인스턴스의 개수만큼 동적 프록시 생성 로직을 작성해야 하므로, 설정 파일이 너무 많이 필요하다.
  2. 컴포넌트 스캔을 활용하여 자동 등록되는 빈에는 프록시 패턴을 활용한 프록시 적용이 불가능하다.
* 이러한 단점은 프록시 팩토리를 실무에서 활용하는 것을 어렵게 하는 걸림돌이 되며, 이를 해결하기 위해서는 빈 후처리기라는 개념을 이해할 필요가 있다.

## 2024-10-22 Tue
### 빈 후처리기란?
* `@Bean` 등의 방식을 활용하여 스프링 빈을 등록할 경우, 스프링은 해당 객체를 직접 생성하여 관리한다.
  * 정확히는 **스프링 컨테이너 내부에 위치한 빈 저장소에 등록한 후, 스프링 컨테이너를 통해 이를 조회할 수 있도록 지원**한다.
* 이러한 과정에서 **스프링이 빈 저장소에 등록하기 위해 자동 생성한 객체를 등록 직전에 조작하고자 하는 경우, 빈 후처리기를 고려**할 수 있다.
  * 이러한 것을 `BeanPostProcessor`라고 지칭하며, 직역에서 알 수 있듯이 빈을 생성한 후 무언가를 처리하기 위한 용도에 사용할 수 있다.
  * 빈 후처리기로 가능한 기능은 대체로 막강하며, 예를 들어 객체 자체를 조작하거나 다른 객체로 바꿔치는 것이 가능하다.

## 2024-10-23 Wed
### 빈 후처리기의 동작 과정
* 스프링이 빈을 등록하는 과정에 빈 후처리기가 개입하는 경우, 동작 과정은 크게 다음과 같은 흐름을 띈다.
  1. 생성: 스프링은 빈 대상이 되는 객체를 생성하며, 이 경우 `@Bean` 어노테이션이 할당된 대상과 컴포넌트 스캔으로 조회된 대상이 모두 생성된다.
  2. 전달: **생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달**한다.
  3. 처리: 빈 후처리기는 자신에게 전달된 스프링 빈 객체를 조작하거나, 다른 객체로 바꿔치는 등의 동작을 적용한다.
  4. 등록: 빈 후처리기는 동작을 마무리한 후 해당 빈 객체를 반환하며, 이렇게 반환된 객체가 빈 저장소에 등록된다.
* 이 때, 4.의 **등록 과정에서 반환된 빈은 처음에 전달된 원본 객체일 수도 있는 반면 빈 후처리기에 의해 교체된 객체일 수도 있다**.

## 2024-10-24 Thu
### BeanPostProcessor 인터페이스 살펴보기
* 빈 후처리기를 구현하기 위해서는 다음과 같이 정의된 `BeanPostProcessor` 인터페이스를 구현한 후 이를 스프링 빈으로 등록해야 한다.
```java
public interface BeanPostProcessor {
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException
}
```
* 이 때, 각 메소드는 다음과 같은 동작을 의미한다.
  * `postProcessBeforeInitialization`: 객체 생성 이후에 `@PostConstructor`와 같은 초기화가 발생하기 전에 호출되는 후처리기를 의미한다.
  * `postProcessAfterInitialization`: 객체 생성 이후에 `@PostConstructor`와 같은 초기화가 발생한 후에 호출되는 후처리기를 의미한다.

## 2024-10-25 Fri
### 스프링의 빈 등록 과정
* 스프링이 빈을 등록하는 과정을 코드로 표현할 경우, 다음과 같이 간단한 코드로 작성해볼 수 있다.
  * 이 경우, `@Configuration` 클래스가 `ApplicationContext`에 등록되므로 클래스 `A`만이 스프링 빈으로 등록된다.
```kotlin
class BasicTest {
    @Test
    fun basicConfig() {
        val context: ApplicationContext = AnnotationConfigApplicationContext(BasicConfig::class.java)

        // A는 빈으로 등록되어 있다.
        val a: A = context.getBean("beanA", A::class.java)
        a.hello()

        // B는 빈으로 등록되지 않는다.
        Assertions.assertThrows(NoSuchBeanDefinitionException::class.java) {
            context.getBean(B::class.java)
        }
    }

    @Configuration
    class BasicConfig {
        @Bean(name = ["beanA"])
        fun a(): A = A()
    }

    class A {
        fun hello() {
            println("hello from A")
        }
    }

    class B {
        fun hello() {
            println("hello from B")
        }
    }
}
```

## 2024-10-26 Sat
### BeanPostProcessor를 활용한 빈 바꿔치기
* 상슬한 로직과 같은 빈 등록 흐름에서 빈 후처리기를 활용할 경우, 등록되는 빈을 바꿔치기하는 것이 가능하다.
  * 이 경우, 상술한 로직의 예로 들어 등록된 빈의 이름은 여전히 `beanA`이나 실제 객체는 클래스 A가 아니게 된다.
* **빈 후처리기를 활용하기 위해서는 단순히 `BeanPostProcessor`를 구현하는 구현체를 다음과 같이 빈으로 등록**하기만 하면 된다. 
```kotlin
class MyPostProcessor : BeanPostProcessor {
    override fun postProcessAfterInitialization(bean: Any, beanName: String): Any? {
        println("beanName: $beanName, bean: $bean")
        return if(bean is A) {
            B()
        } else {
            bean
        }
    }
}
```
* 이렇게 구현되어 등록된 **빈 후처리기는 내부적으로 높은 우선 순위를 갖기에 다른 빈들이 등록되기 전에 동작하므로 후처리의 역할을 담당**할 수 있다.

## 2024-10-27 Sun
### BeanPostProcessor의 동작 확인
* 상술한 빈 후처리기의 동작을 확인하기 위해서는 다음과 같은 테스트 코드를 작성해볼 수 있다.
```kotlin
class BeanPostProcessorTest {
    @Test
    fun basicConfig() {
        val context: ApplicationContext = AnnotationConfigApplicationContext(BeanPostProcessorConfig::class.java)

        // B가 빈으로 등록되어 있다.
        val a: B = context.getBean("beanA", B::class.java)
        a.hello()

        // 이제는 A가 빈으로 등록되지 않는다.
        Assertions.assertThrows(NoSuchBeanDefinitionException::class.java) {
            context.getBean(A::class.java)
        }
    }
}
```
* 이 경우, 등록된 빈 후처리기인 `MyPostProcessor` 클래스의 내부 동작 과정에 의해 아래와 같은 로그가 출력된다.
  * 이를 통해 빈 후처리기에 전달된 빈은 `BeanPostProcessorTest$A`였으나, 실제로 반환된 것은 클래스 `B`임을 확인할 수 있다.
  * 즉, 빈 후처리기의 동작 방식으로 인해 `beanA` 이름으로 클래스 `B`가 등록되며 `@Bean` 어노테이션이 있는 `A`는 빈에 등록되지 않는 점을 알 수 있다.
```
beanName: beanA, bean: ga.injuk.spring.study.advanced.proxy.postprocessor.BeanPostProcessorTest$A@4315e9af
```

## 2024-10-28 Mon
### 빈 후처리기의 이점
* **빈 후처리기는 빈을 조작하고 변경할 수 있는 일종의 후킹 포인트이며, 상술한 바와 같이 빈 자체를 바꿔버릴 수 있다는 점에서 매우 강력**하다.
* 컴포넌트 스캔에 의해 스프링 컨테이너에 빈으로 등록되는 객체들은 일반적으로 중도에서 가로채어 조작할 수 있는 방법이 없으나, 빈 후처리기는 이를 가능케 한다.
  * 이는 즉 **개발자가 명시적으로 등록한 모든 빈을 중간에서 조작할 수 있음을 의미하며, 심지어는 빈 객체를 프록시로 교체하는 것까지도 가능**하다.

## 2024-10-29 Tue
### 프록시 적용 여부 조절하기
* 스프링의 경우 개발자 **스스로가 등록한 빈 뿐만 아니라 스프링 내부적으로 자동 등록되는 빈이 굉장히 많으며, 이들 모두가 빈 후처리기로 전달**된다.
  * 이 때, **스프링이 제공하는 빈 중에는 프록시를 생성할 수 없는 빈도 많기에 모든 빈에 대해 일괄적인 후처리 로직을 적용하면 예외가 발생하기 쉽다**.
* 때문에 어떤 빈에 프록시를 적용할지 조절하는 것은 굉장히 중요하며, 이를 위해 패키지 명 등을 활용해볼 수 있다.
  * 그러나 더 **좋은 방법은 이미 제공되는 기능인 클래스와 메소드 단위의 필터링 기능을 포함하는 포인트 컷을 활용하는 것**이다.
* 결과적으로 포인트 컷은 크게 다음과 같은 용도로 사용됨을 이해할 수 있다.
  1. 빈 후처리기와 프록시 자동 생성: 프록시 적용 대상 여부를 체크하여 꼭 필요한 곳에만 프록시를 적용할 수 있다.
  2. 프록시 내부로부터의 활용: 프록시의 어떤 메소드가 호출되었을 때 어드바이스를 적용할지 여부를 판단할 수 있다.

## 2024-10-30 Wed
### 프록시 생성의 한계와 빈 후처리기
* `@Configuration`을 활용하여 프록시를 직접 등록하는 것은 프록시와 관련된 코드가 너무 많아진다는 단점이 있다.
  * 가뜩이나 많은 빈을 쉽게 관리하기 위해 `@ComponentScan`까지 활용하는데, 이러한 방식을 활용조차 못하게 할 수 있기에 득보다 실이 많을 수 있다.
* **빈 후처리기를 활용할 경우 프록시 생성 로직을 가로채어 원본 대신 프록시를 스프링 빈으로 등록할 수 있으며, 컴포넌트 스캔 역시 활용이 가능**하다.
* 심지어 **스프링은 이러한 빈 후처리기조차 이미 만들어 제공하고 있으며, 개발자는 이를 그저 가져다 활용하기만 해도 무방**하다.