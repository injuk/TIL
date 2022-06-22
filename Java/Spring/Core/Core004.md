# Core
## 2022-06-23 Thu

### 스프링 컨테이너 생성하기
* 스프링 컨테이너는 다음과 같이 생성해볼 수 있다.
```
ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
```
* 상술한 코드에서 생성된 `ApplicationContext`가 스프링 컨테이너이다.
  * **`ApplicationContext`는 인터페이스이며, `AnnotationConfigApplicationContext`는 구현체**이다.
* 스프링 컨테이너를 직접 생성하는 경우, 다음과 같은 방식 중 하나로 생성해볼 수 있으며, 상술한 방식은 2.의 방식을 따라 생성한 경우이다.
  1. XML을 기반으로 생성하기
  2. 어노테이션 기반의 Java 설정 클래스로 생성하기
* 스프링 자체가 어노테이션을 적극적으로 활용하고 지원하는 프레임워크다보니, 최근에는 XML을 기반으로 하는 방식은 잘 사용되지 않는다.
* 더 정확하게는 스프링 컨테이너는 `ApplicationContext`와 `BeanFactory`로 구분할 수 있다. 
  * **일반적으로는 둘 중 더 자주 사용되는 `ApplicationContext`를 스프링 컨테이너**라고 한다.

### 스프링 컨테이너 생성 과정
* 스프링 컨테이너를 생성하기 위해서는 구성 정보를 전달해줄 필요가 있다.
  * **스프링 컨테이너는 전달된 구성 정보를 토대로 빈으로 등록할 객체를 결정**한다.
  * 상술한 예시에서, `new AnnotationConfigApplicationContext(AppConfig.class)`는 AppConfig를 구성 정보로 전달한다.
  * **이를 통해 스프링 컨테이너가 생성되며, 생성된 스프링 컨테이너는 내부적으로 스프링 빈 저장소를 포함**한다.
  * **스프링 빈 저장소는 키 - 값 목록으로 구성되며, 이 경우 키와 값은 각각 빈 이름과 빈 객체에 대응**된다. 
* 스프링 컨테이너가 생성되면 스프링 빈 저장소에 필요한 빈 객체를 등록한다.
  * 이 때, **스프링 컨테이너는 매개변수로 전달된 설정 클래스의 정보 중 @Bean 어노테이션이 명시된 메소드를 모두 호출하여 스프링 빈을 등록**한다.
  * 등록되는 빈 이름은 기본적으로 @Bean 어노테이션이 명시된 메소드의 이름을 사용하며, 필요시 `@Bean(name="ingnoh")`와 같이 직접 작성할 수도 있다.
    * **빈 이름은 반드시 유일한 이름을 지정해야하며, 그렇지 않은 경우 임의의 빈이 무시되는 등 예상치 못한 오류가 발생**할 수 있다.
    * 특히 실무에서의 빈 이름은 최대한 단순하고 명확해야하며, 절대 중복된 이름은 명명하지 말아야 한다.
  * 즉, **스프링 빈 저장소에 저장되는 빈 이름은 호출되는 메소드명이며, 해당 빈 이름에 대응되는 빈 객체에는 메소드로부터 반환된 값이 저장**된다. 
* **스프링 컨테이너가 생성된 후 필요한 빈 객체 정보를 관리하도록 구성이 완료된 스프링 빈 저장소를 스프링 빈이라고 지칭**할 수 있다.
* 여기까지 과정에서 스프링 빈이 준비되었으므로, 각 빈의 의존 관계를 설정한다.
  * **스프링 컨테이너는 설정 클래스의 정보를 기반으로 의존 관계를 주입**한다.
* **스프링은 빈 객체를 생성하고 의존 관계를 주입하는 단계가 명시적으로 구분되어 있다**.
  * 반면, AppConfig와 같이 Java 코드만으로 의존성을 주입하는 경우에는 두 단계가 구분되지 않는다.

### 컨테이너에 등록된 빈 조회하기
* **`ac.getBeanDefinitionNames()`를 통해 스프링 컨테이너에 등록된 모든 빈의 이름을 조회**할 수 있다.
  * 조회된 각 빈의 이름은 일반적으로 설정 파일로 전달된 클래스의 메소드이름이다.
* **`ac.getBean()`을 통해 빈 이름을 기준으로 대응되는 빈 객체를 조회**할 수 있다.
* 상술한 내용을 토대로 컨테이너에 등록한 빈을 모두 조회할 경우, 다음과 같은 테스트 코드를 활용할 수 있다.
  * getBeanDefinitionNames 메소드를 통해 모든 bean의 definitionName을 가져올 수 있다.
  * 가져온 배열을 순회하며, getBean 메소드를 통해 beanDefinitionName에 대응되는 빈 객체를 가져올 수 있다.
  * 이 경우, 스프링이 내부적인 동작을 위해 등록한 빈까지 모두 출력된다.
```
@Test
void findAllBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
        Object bean = ac.getBean(beanDefinitionName);
        System.out.println("name = " + beanDefinitionName + " object = " + bean);
    }
}
```
* 스프링이 내부적인 동작을 위해 등록한 빈을 제외하고 확인하고 싶은 경우, 다음과 같은 테스트 코드를 활용할 수 있다.
  * 빈의 종류를 분류하기 위해 beanDefinition의 getRole 메소드를 활용할 수 있다.
  * **`ROLE_APPLICATION`은 스프링이 내부적인 동작을 위해 등록한 빈이 아닌, 애플리케이션 개발을 위해 개발자가 등록한 빈을 의미**한다.
  * 반면, `ROLE_INFRASTRUCTURE`는 스프링이 내부적인 동작을 위해 자동으로 등록하는 빈을 의미한다.
```
@Test
void findApplicationBean() {
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for (String beanDefinitionName : beanDefinitionNames) {
        BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

        if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + " object = " + bean);
        }
    }
}
```