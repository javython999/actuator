# 액츄에이터
## 프로덕션 준비 기능이란?
개발자가 애플리케이션을 개발할 때는 기능 요구사항만 개발하는 것이 아니다.
서비스를 실제 운영 단계에 올리게 되면 개발자들이 해야하는 또 다른 중요한 업무가 있다.
바로 서비스에 문제가 없는지 모니터링하고 지표들을 심어서 감시하는 활동들이다.

운영 환경에서 서비스할 때 필요한 이런 기능들을 프로덕션 준비 기능이라 한다.
쉽게 이야기해서 프로덕션을 운영에 배포할 때 준비해야하는 비 기능적 요소들을 뜻한다.
* 지표(metric), 추적(trace), 감사(auditing)
* 모니터링

좀 더 구체적으로 설명하자면, 애플리케이션이 현재 살아있는지, 로그 정보는 정상 설정 되었는지,
커넥션 풀은 얼마나 사용되고 있는지 등을 확인할 수 있어야 한다.

스프링 부트가 제공하는 액추에이터는 이런 프로덕션 준비 기능을 매우 편리하게 사용할 수 있는 다양한 편의 기능들을 제공한다.
더 나아가 마이크로미터, 프로메테우스, 그라파나 같은 최근 유행하는 모니터링 시스템과 매우 쉽게 연동할 수 있는 기능도 제공한다.

## 액츄에이터 시작
액추에이터가 제공하는 프로덕션 준비 기능을 사용하려면 스프링 부트 액츄에이터 라이브러리를 추가해야 한다.

build.gradle
```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```
동작 확인
* 기본 메인 클래스 실행
* http://localhost:8080/actuator

액츄에이터 기능을 웹에 노출
application.yml
```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```
각각의 엔드포인트는 `/actuator/{엔드포인트명}`과 같은 형식으로 접근할 수 있다.
* `http://localhost:8080/actuator/health`: 애플리케이션 헬스 정보를 보여준다.
* `http://localhost:8080/actuator/beans`: 스프링 컨테이너에 등록된 빈을 보여준다.

## 엔드포인트 설정
엔드포인트를 사용하려면 다음 2가지 과정이 모두 필요하다.
1. 엔드포인트 활성화
2. 엔드포인트 노출

엔드포인트를 활성화 한다는 것은 해당 기능 자체를 사용할지 말지 `on`, `off`를 선택하는 것이다.
엔드포인트를 노출하는 것은 활성화된 엔드포인트를 HTTP에 노출할지 아니면 JMX에 노출할지 선택하는 것이다.
엔드포인트는 대부분 기본으로 활성화 되어있다.(`shutdown` 제외)노출이 되어있지 않을 뿐이다.

application.yml - 모든 엔드포인트를 웹에 노출
```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"
```
엔드포인트 노출
```yml
management:
  endpoints:
    jmx:
      exposure:
        include: "health,info"
```
* `jmx`에 `health, info`를 노출한다.

```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"
        exclude: "env, beans"
```
* `web`에 모든 엔드포인트를 노출하시만 `env`, `beans`는 제외한다.

## 다양한 엔드포인트
각각의 엔드포인트를 통해서 개발자는 애플리케이션 내부의 수 많은 기능을 관리하고 모니터링 할 수 있다.
스프링 부트가 기본으로 제공하는 다양한 엔드포인트에 대해서 알아보자. 다음은 자주 사용하는 기능 위주로 정리했다.

엔드포인트 목록
* `beans`: 스프링 컨테이너에 등록된 스프링 빈을 보여준다.
* `conditions`: `condition`을 통해서 빈을 등록할 때 평가 조건과 일치하거나 일치하지 않은 이유를 표시한다.
* `configprops`: `@ConfigurationProperties`를 보여준다.
* `env`: `Environment` 정보를 보여준다.
* `health`: 애플리케이션 헬스 정보를 보여준다.
* `httpexchanges`: HTTP 호출 응답 정보를 보여준다. `HttpExchangeRepository`를 구현한 빈을 별도로 등록해야 한다.
* `info`: 애플리케이션 정보를 보여준다.
* `loggers`: 애플리케이션 로거 설정을 보여주고 변경도 할 수 있다.
* `metrics`: 애플리케이션의 메트릭 정보를 보여준다.
* `mappings` `@ReqeustMapping` 정보를 보여준다.
* `threaddump` 쓰레드 덤프를 실행해서 보여준다.
* `shutdown`: 애플리케이션을 종료한다. 이 기능은 기본으로 비활성화 되어있다.

### 헬스 정보
헬스 정보를 사용하면 애플리케잇연에 문제가 발생했을 때 문제를 빠르게 인지할 수 있다.
`http://localhost:8080/actuator/health`

기본 동작
```json
{"status": "UP"}
```
헬스 정보는 단순히 애플리케이션이 요청에 응답을 할 수 있는지 판단하는 것을 넘어서 애플리케이션이 사용하는 데이터베이스가 응답하는지, 디스크 사용량에는 문제가 없는지 같은 다양한 정보들을 포함해서 만들어진다.
헬스 정보를 더 자세히 보려면 다읍 옵션을 지정하면 된다.
```yml
management:
  endpoint:
    health:
      show-details: always
```

show-details 옵션
```json
{
 "status": "UP",
 "components": {
 "db": {
 "status": "UP",
 "details": {
 "database": "H2",
 "validationQuery": "isValid()"
      }
    },
 "diskSpace": {
 "status": "UP",
 "details": {
 "total": 994662584320,
 "free": 303418753024,
 "threshold": 10485760,
 "path": ".../spring-boot/actuator/actuator/.",
 "exists": true
      }
    },
 "ping": {
 "status": "UP"
    }
  }
}
```
각각의 항목이 아주 자세하게 노출되는 것을 확인할 수 있다.
이렇게 자세하게 노출하는 것이 부담스럽다면 `show-details` 옵션을 제거하고 대신 다음 옵션을 사용하면 된다.
```yml
management:
  endpoint:
    health:
      show-components: always
```

## 애플리케이션 정보
`info` 엔드포인트는 애플리케이션의 기본 정보를 노출한다.

기본으로 제공하는 기능들은 다음과 같다.
* `java`: 자바 런타임 정보
* `os`: OS 정보
* `env`: `Environment`에서 `info.`로 시작하는 정보
* `build`: 빌드 정보
* `git`: `git` 정보, `git.properties` 파일이 필요하다.

## 로거
`loggers` 엔드포인트를 사용하면 로깅과 관련된 정보를 확인하고, 또 실시간으로 변경할 수도 있다.
코드를 통해서 알아보자.

application.yml
```yml
logging:
  level:
    hello.controller: debug
```
`hello.controller` 패키지와 그 하위는 `debug` 레벨을 출력하도록 했다.

* 로그를 별도로 설정하지 않으면 스프링 부트는 기본으로 `INFO`를 사용한다. 실행결과를 보면 `ROOT`의 `configuredLevel`가 `INFO`인 것을 확인할 수 있다. 따라서 그 하위도 모두 `INFO` 레벨이 적용된다.

더 자세히 조회하기   
다음과 같은 패턴을 사용해서 특정 로거 이름을 기준으로 조회할 수 있다.    
`http://localhost:8080/actuator/loggers/{로거이름}`

### 실시간 로그 레벨 변경
`loggers` 엔드포인트를 사용하면 애플리케이션을 다시 시작하지 않고, 실시간으로 로그 레벨을 변경할 수 있다.
Postman 같은 프로그램으로 POST 요청을 해보자.     
`POST http://localhost:8080/actuator/loggers/hello.controller`

POST로 전달하는 내용 JSON
```json
{
  "configuredLevel": "TRACE"
}
```
## HTTP 요청 응답 기록
HTTP 요청과 응답의 과거 기록을 확인하고 싶다면 `httpexchanges` 엔드포인트를 사용하면 된다.
`HttpExchangeRepository` 인터페이스를 구현체를 빈으로 등록하면 `httpexchanges` 엔드포인트를 사용할 수 있다.
(주의! 해당 빈을 등록하지 않으면 `httpexchanges` 엔드포인트가 활성화 되지않는다.)

## 액츄에이터와 보안
### 보안 주의
액츄에이터가 제공하는 기능들은 우리 애플리케이션의 내부 정보를 너무 많이 노출한다.
그래서 외부 인터넷 망이 공개된 곳에 액츄에이터의 엔드포인트를 공개하는 것은 보안상 좋은 방안이 아니다.
액츄에이터의 엔드포인트들은 외부 인터넷에서 접근이 불가능하게 막고, 내부에서만 접근 가능한 내부망을 사용하는 것이 안전하다.

#### 액츄에이터를 다른 포트에서 실행
예를 들어서 외부 인터넷 망을 통해서 8080 포트에만 접근할 수있고, 다른 포트는 내부망에서만 접근할 수 있다면 액츄에이터에 다른 포트를 설정하면 된다.
액츄에이터의 기능을 애플리케이션 서버와느 ㄴ다른 포트에서 실행하려면 다음과 같이 설정하면된다. 이 경우 기존 8080포트에서는 액츄에이터를 접근할 수 없다.

#### 액츄에이터 포트 설정
```yml
management:
  server:
    port: 9292
```

#### 액츄에이터 URL 경로에 인증 설정
포트를분리하는 것이 어렵고 어쩔 수 없이 외부 인터넷 망을 통해서 접근해야 한다면 `/actuator` 경로에 서블릿 필터, 또는 스프링 시큐리티를 통해서 인증된 사용자만 접근 가능하도록 추가 개발이 필요하다.

### 엔드포인트 경로 변경
엔드포인트의 기본 경로를 변경하려면 다음과 같이 설정하면 된다.
```yml
management:
  endpoints:
    web:
      base-path: "/manage"
```