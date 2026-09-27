# Spring Core

Spring을 활용하여 **회원·주문·할인 시스템**을 구현하며  
객체 지향 설계 원칙과 Spring의 핵심 원리인 **IoC, DI, Spring Container, Bean**을 학습한 프로젝트입니다.

## 주요 기능

- 회원 가입 및 조회
- BASIC / VIP 회원 등급 관리
- 주문 생성
- 회원 등급에 따른 할인 적용
- 고정 금액 할인 정책 구현
- 정률 할인 정책 구현
- 할인 정책 교체가 가능한 구조 설계
- Spring Container를 활용한 객체 및 의존관계 관리

## 프로젝트 구조

```text
MemberService
      ↓
MemberRepository
      ↓
MemoryMemberRepository


OrderService
   ├── MemberRepository
   │       ↓
   │  MemoryMemberRepository
   │
   └── DiscountPolicy
           ├── FixDiscountPolicy
           └── RateDiscountPolicy
```

구현체가 아닌 인터페이스에 의존하도록 설계하고,  
Spring Container가 실제 구현 객체를 생성하고 의존관계를 주입하도록 구성했습니다.

## 기술 스택

### Backend
- Java
- Spring Boot
- Spring Core

### Test
- JUnit 5
- AssertJ

### Build
- Gradle

## 구현 및 학습 내용

### 1. 객체 지향 설계

회원, 주문, 할인 도메인을 구현하며 객체 지향 설계 원칙을 적용했습니다.

```text
회원
├── Member
├── MemberService
└── MemberRepository

주문
└── OrderService

할인
├── DiscountPolicy
├── FixDiscountPolicy
└── RateDiscountPolicy
```

역할과 구현을 분리하고 인터페이스를 활용하여  
구현체 변경이 비즈니스 로직에 미치는 영향을 줄이는 구조를 학습했습니다.

### 2. 할인 정책 변경

VIP 회원을 대상으로 두 가지 할인 정책을 구현했습니다.

```text
DiscountPolicy
      ├── FixDiscountPolicy
      │       └── 고정 금액 할인
      │
      └── RateDiscountPolicy
              └── 주문 금액의 일정 비율 할인
```

할인 정책을 변경하는 과정에서 클라이언트가 인터페이스뿐만 아니라  
구현 클래스에도 의존할 경우 DIP와 OCP를 위반할 수 있음을 확인했습니다.

### 3. AppConfig와 관심사의 분리

객체 생성과 연결에 대한 책임을 `AppConfig`로 분리했습니다.

```text
AppConfig
   │
   ├── MemberService
   │       └── MemberRepository
   │
   └── OrderService
           ├── MemberRepository
           └── DiscountPolicy
```

서비스 객체는 자신의 비즈니스 로직에 집중하고,  
객체 생성과 의존관계 구성은 외부 설정이 담당하도록 구성했습니다.

이를 통해 **관심사의 분리와 DIP, OCP**를 적용했습니다.

### 4. IoC / DI

객체가 직접 필요한 구현체를 생성하지 않고  
외부에서 구현 객체를 생성하고 연결하도록 구조를 변경했습니다.

```text
기존

OrderService
     ↓
new RateDiscountPolicy()


변경

AppConfig
   ↓
OrderService
   ↓
DiscountPolicy
```

- **IoC (Inversion of Control)** : 객체 생성과 제어의 책임을 외부로 분리
- **DI (Dependency Injection)** : 객체가 필요로 하는 의존관계를 외부에서 주입

### 5. Spring Container & Bean

`ApplicationContext`를 사용하여 Spring Container를 생성하고  
객체를 Spring Bean으로 등록하여 관리했습니다.

```java
ApplicationContext applicationContext =
        new AnnotationConfigApplicationContext(AppConfig.class);
```

Spring Container가 설정 정보를 기반으로

```text
설정 정보 확인
      ↓
Spring Bean 등록
      ↓
의존관계 설정
      ↓
Bean 관리
```

과정을 수행하는 구조를 학습했습니다.

또한 Bean 이름과 타입을 활용한 조회 및  
동일한 타입의 Bean이 여러 개 존재하는 경우의 조회 방법을 학습했습니다.

### 6. Singleton Container

Spring Container가 기본적으로 객체를 **Singleton**으로 관리하는 원리를 학습했습니다.

```text
Client A ─┐
Client B ─┼──→ Spring Container ──→ Singleton Bean
Client C ─┘
```

여러 요청마다 객체를 새로 생성하지 않고  
하나의 객체를 생성하여 공유함으로써 효율적으로 객체를 관리합니다.

Singleton 객체는 여러 클라이언트가 공유하므로  
상태를 유지하는 필드를 가지지 않는 **Stateless 설계**가 중요하다는 점을 학습했습니다.

### 7. Component Scan

`@ComponentScan`을 사용하여 Spring Bean을 자동으로 탐색하고 등록했습니다.

```java
@Configuration
@ComponentScan
public class AutoAppConfig {
}
```

각 구현 클래스에 `@Component`를 적용하여  
직접 `@Bean`을 등록하지 않아도 Spring Container가 자동으로 Bean을 등록하도록 구성했습니다.

```java
@Component
public class MemberServiceImpl implements MemberService {
}
```

### 8. Dependency Injection

Spring의 다양한 의존관계 주입 방법을 학습했습니다.

```text
Constructor Injection
Setter Injection
Field Injection
Method Injection
```

그중 필수 의존관계를 명확하게 표현하고  
불변성을 유지할 수 있는 **생성자 주입**을 중심으로 적용했습니다.

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

    public OrderServiceImpl(
            MemberRepository memberRepository,
            DiscountPolicy discountPolicy) {

        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

생성자가 하나인 경우 `@Autowired`를 생략해도  
Spring이 자동으로 의존관계를 주입하는 방식도 확인했습니다.

### 9. 여러 Bean의 의존관계 해결

동일한 타입의 Spring Bean이 여러 개 존재할 때 발생하는 문제와  
이를 해결하는 방법을 학습했습니다.

```text
@Autowired
@Qualifier
@Primary
```

또한 `List`, `Map`을 활용하여 동일한 타입의 Bean을 모두 조회하고  
필요한 구현체를 동적으로 선택하는 방법을 학습했습니다.

### 10. Bean Life Cycle

Spring Bean의 생성부터 소멸까지의 생명주기를 학습했습니다.

```text
Spring Container 생성
        ↓
Bean 생성
        ↓
Dependency Injection
        ↓
초기화 Callback
        ↓
Bean 사용
        ↓
소멸 Callback
        ↓
Spring 종료
```

초기화와 종료 작업을 처리하기 위해 다음 방법을 학습했습니다.

```text
InitializingBean / DisposableBean

@Bean(
    initMethod = "...",
    destroyMethod = "..."
)

@PostConstruct
@PreDestroy
```

객체 생성과 초기화의 책임을 분리하고  
의존관계 주입이 완료된 이후 초기화 로직을 수행하도록 구성했습니다.

### 11. Bean Scope

Spring Bean이 존재할 수 있는 범위인 Bean Scope를 학습했습니다.

```text
Singleton
Prototype
Request
Session
Application
```

#### Singleton

```text
Spring Container 시작
        ↓
Bean 생성
        ↓
동일한 Bean 공유
        ↓
Spring Container 종료
```

#### Prototype

```text
Bean 요청
   ↓
새로운 Bean 생성
   ↓
Dependency Injection
   ↓
초기화
   ↓
Client에게 반환
```

Prototype Scope에서는 Bean을 요청할 때마다 새로운 객체가 생성되며,  
Spring Container는 생성과 의존관계 주입, 초기화까지만 관리합니다.

또한 Singleton Bean과 Prototype Bean을 함께 사용할 때 발생할 수 있는 문제와  
Provider를 활용한 해결 방법을 학습했습니다.

### 12. Web Scope

웹 환경에서 사용하는 Scope의 특징을 학습했습니다.

```text
request
    └── HTTP 요청 하나의 생명주기

session
    └── HTTP Session의 생명주기

application
    └── ServletContext의 생명주기
```

Request Scope를 직접 적용하고  
Provider와 Proxy를 활용하여 Scope가 다른 Bean 사이의 의존관계를 처리하는 방법을 학습했습니다.

## 핵심 학습 내용

- 객체 지향 설계와 SOLID 원칙
- 역할과 구현의 분리
- OCP / DIP
- IoC와 DI
- Spring Container
- Spring Bean
- Singleton Container
- Stateless 설계
- Component Scan
- `@Autowired`
- Constructor Injection
- `@Qualifier` / `@Primary`
- Bean Life Cycle
- `@PostConstruct` / `@PreDestroy`
- Bean Scope
- Prototype Scope
- Web Scope
- Provider / Proxy
