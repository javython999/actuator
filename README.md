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
---
# 마이크로미터, 프로메테우스, 그라파나
## 마이크로미터
세상에는 수 많은 모니터링 툴이 있고, 시스템의 다양한 정보를 이 모니터링 툴에 전달해서 사용하게 된다.
그런데 중간에 사용하는 모니터링 툴을 변경하게 되면 어떻게 될까?
기존에 측정했던 코드를 모두 변경한 툴에 맞도록 다시 변경해야 한다.
개발자 입장에서는 단순히 툴 하나를 변경했을 뿐인데, 측정하는 코드까지 모두 변경해야 하는 문제가 발생한다.
이런 문제를 해결하는 것이 바로 마이크로미터(Micrometer)라는 라이브러리이다.

* 마이크로미터는 애플리케이션 메트릭 파사드라고 불리는데, 애플리케이션의 메트릭(측정 지표)을 마이크로미터가 정한 표준 방법으로 모아서 제공해준다.
* 마이크로미터가 추상화를 통해서 구현체를 쉽게 갈아끼울 수 있도록 해준다.
* 보통은 스프링이 이런 추상화를 직접 만들어서 제공하지만, 마이크로미터라는 이미 잘 만들어진 추상화가 있기 때문에 스프링은 이것을 활용한다.
* 스프링 부트 액츄에이터는 마이크로미터를 기본으로 내장해서 사용한다.

## 메트릭 확인하기
CPU, JVM, 커넥션 사용 등등 수 많은 지표들을 어떻게 수집해야 할까?
개발자가 각각의 지표를 직접 수집해서 그것을 마이크로미터가 제공하는 표준 방법에 따라 등록하면 된다.
다행히도 마이크로미터는 다양한 지표 수집 기능을 이미 만들어서 제공한다.
그리고 스프링 부트 액츄에이터는 마이크로미터가 제공하는 지표 수집을 `@AutoConfiguration`을 통해서 자동으로 등록해준다.

### metrics 엔드포인트
`metrics` 엔드포인트를 사용하면 기본으로 제공되는 메티릭들을 확인할 수 있다.
`metrics` 엔드포인트는 다음과 같은 패턴을 사용해서 더 자세히 확인할 수있다.
`http://localhost:8080/actuator/meetrics/{name}`

### tag 필터
`avaliableTags`를 보면 다음과 같은 항목을 확인할 수 있다.
* `tag:area`, `values[heap, nonheap]`
* `tag:id`, `values[G1 Survivor Space ...]`

해당 Tag를 기반으로 정보를 필터일해서 확인할 수 있다.
`tage=KET:VALUE` 형식으로 사용해야 한다.

* `http://localhost:8080/actuator/metrics/jvm.memory.used?tag=area:heap`

## 다양한 메트릭
마이크로미터와 액츄에이터가 기본으로 제공하는 다양한 메트릭을 확인해보자.
* JVM 메트릭
* 시스템 메트릭
* 애플리케이션 시작 메트릭
* 스프링 MVC 메트릭
* 톰캣 메트릭
* 데이터 소스 메트릭
* 로그 메트릭
* 기타 수 많은 메트릭이 있다.
* 사용자가 메트릭을 직접 정의하는 것도 가능하다.

### JVM 메트릭
JVM과 과련된 메트릭을 제공한다. `jvm.`으로 시작한다.
* 메모리 및 버퍼 풀 세부 정보
* 가비지 수집 관련 통계
* 스레드 활용
* 로드 및 언로드된 클래스 수
* JVM 버전 정보
* JIT 컴파일 시간

### 시스템 메트릭
시스템 메트릭을 제공한다. `system.`, `process.`, `disk.`으로 시작한다.
* CPU 지표
* 파일 디스크립터 메트릭
* 가동 시간 메트릭
* 사용 가능한 디스크 공간

### 애플리케이션 시작 메트릭
애플리케이션 시작 시간 메트릭을 제공한다.
* `application.started.time`: 애플리케이션을 시작하는데 걸리는 시간(`ApplicationStartedEvent`로 측정)
* `application.ready.time`: 애플리케이션이 요청을 처리할 준비가 되는데 걸리는 시간(`ApplicationReadyEvent`로 측정)

스프링 내부에 여러 초기화 단계가 있고 각 단계별로 내부에서 애플리케이션 이벤트를 발행한다.
* `ApplicationStartedEvent`: 스프링 컨테이너가 완전히 실행된 상태이다. 이후 커맨드 라인 러너가 호출 된다.
* `ApplicationReadyEvent`: 커맨드 라인 러너가 실행된 이후에 호출 된다.

### 스프링 MVC 메트릭
스프링 MVC 컨트롤러가 처리하는 모든 요청을 다룬다.
메트릭 이름: `http.server.requests`

`TAG`를 사용해서 다음 정보를 분류해서 확인할 수 있다.
* `uri`: 요청 URI
* `method`: `GET`, `POST` 같은 HTTP 메서드
* `status`: `200`, `400`, `500` 같은 HTTP Status 코드
* `exception`: 예외
* `outcome`: 상태코드를 그룹으로 모아서 확인 `1xx:INFORMATIONAL`, `2xx:SUCCESS`, `3xx:REDIRECTION`, `4xx:CLIENT_ERROR`, `5xx:SERVER_ERROR`

### 데이터소스 메트릭
`DataSource`, 커넥션 풀에 관한 메트릭을 확인할 수 있다.
`jdbc,connections.`으로 시작한다.
최대 커넥션, 최소 커넥션, 활성 커넥션, 대기 커넥션 수 등을 확인할 수 있다.
히카리 커넥션 풀을 사용하면 `hikaricp.`를 통해 히카리 커넥션 풀의 자세한 메트릭을 확인할 수 있다.

### 로그 메트릭
`logback.events`: logback 로그에 대한 메트릭을 확인할 수 있다.
`trace`, `debug`, `info`, `warn`, `error` 각각의 로그 레벨에 따른 로그 수를 확인할 수 있다.

### 톰캣 메트릭
톰캣 메트릭은 `tomcat.`으로 시작한다.
톰캣 메트릭을 모두 사용하려면 다음 옵션을 켜야 한다.(옵션을 켜지 않으면 `tomcat.session.` 관련 정보만 노출된다.)

application.yml
```yml
server:
  tomcat:
    mbeanregistry:
      enable: true
```
톰캣의 최대 스레드, 사용 스레드 수를 포함한 다양한 메트릭을 확인할 수 있다.

### 기타
* HTTP 클라이언트 메트릭(`RestTemplate`, `WebClient`)
* 캐시 메트릭
* 작업 실행과 스케줄 메트릭
* 스프링 데이터 리포지토리 메트릭
* 몽고DB 메트릭
* 레디스 메트릭

### 사용자 정의 메트릭
사용자가 직접 메트릭을 정의할 수 도 있다. 예를 들어서 주문수, 최소수를 메트릭으로 만들 수 있다.

## 프로메테우스와 그라파나 소개
프로메테우스
애플리케이션에서 발생한 메트릭을 그 순간만 확인하는 것이 아니라 과거 이력까지 함께 확인하려면 메트릭을 보관하는 DB가 필요하다.
이렇게 하려면 어디선가 메트릭을 지속해서 수집하고 DB에 저장해야 한다. 프로메테우스가 바로 이런 역할을 담당한다.

그라파나
프로메테우스가 DB라고 하면, 이 DB에 있는 데이터를 불러서 사용자가 보기 편하게 보여주는 대시보드가 필요하다.
그라파나는 매우 유연하고, 데이터를 그래프로 보여주는 툴이다. 수 많은 그래프를 제공하고, 프로메테우스를 포함한 다양한 데이터 소스를 지원한다.

### 전체 구조
1. 스프링 부트 액츄에이터와 마이크로미터를 사용하면 수 많은 메트릭을 자동으로 생성한다.
    * 마이크로미터 프로메테우스 구현체는 프로메테우스가 읽을 수 있는 포멧으로 메트릭을 생성한다.
2. 프로메테우스는 이렇게 만들어진 메트릭을 지속해서 수집한다.
3. 프로메테우수는 수집한 메트릭을 내부 DB에 저장한다.
4. 사용자는 그라파나 대시보드 툴을 통해서 그래프로 편리하게 메트릭을 조회한다. 이때 필요한 데이터는 프로메테우스를 통해서 조회한다.

## 프로메테우스
### 애플리케이션 설정
프로메테우스는 메트릭을 수집하고 보관하는 DB이다. 프로메테우스가 우리 애플리케이션의 메트릭을 수집하도록 연동해야 한다.
여기에는 2가지 작업이 필요하다.
1. 애플리케이션 설정: 프로메테우스가 애플리케이션의 메트릭을 가져갈 수 있또록 애플리케이션에서 프로메테우스 포맷에 맞추어 메트릭 생성
2. 프로메테우스 설정: 프로메테우스가 애플리케이션의 메트릭을 주기적으로 수집하도록 설정

build.gradle 추가
```groovy
implementation 'io.micrometer:micrometer-registry-prometheus' // prometheus
```
* 마이크로미터 프로메테우스 구현 라이브러리를 추가 한다.
* 이렇게 하면 스프링 부트와 액츄에이터가 자동으로 마이크로 프로메테우스 구현체를 등록해서 동작하도록 설정해준다.
* 액츄에이터에 프로메테우스 메트릭 수집 엔드포인트가 자동으로 추가 된다.
  * `/actuator/prometheus`

### 포맷 차이
* `jvm.info` -> `jvm_info`: 프로메테우스는 `.` 대신 `_` 포맷을 사용한다. `.` 대신에 `_` 포맷으로 변환 된 것을 확인할 수 있다.
* `logback.events` -> `logback_events_total`: 로그 수 처럼 지속해서 숫자가 증가하는 메트릭을 카운터라 한다. 프로메테우스는 카운터 메트릭의 마지막에 관례상 `_total`을 붙인다.
* `http.server.requests` -> 이 메트릭은 내부에 요청 수, 시간 합, 최대 시간 정보를 가지고 있었다. 프로메테우스에서는 다음 3가지로 분리된다.
  * `http_server_requests_seconds_count`: 요청 수
  * `http_server_requests_seconds_sum`: 시간 합(요청 수의 시간을 합함)
  * `http_server_reqeusts_seconds_max`: 최대 시간(가장 오래 걸린 요청 수)

### 프로메테우스 수집 설정
프로메테우스가 애플리케이션의 `/actuator/prometheus`를 호출해서 메트릭을 주기적으로 수집하도록 설정해보자.
프로메테우스 폴더에 있는 `prometheus.yml` 파일을 수정하자.

prometheus.yml
```yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
alerting:
  alertmanagers:
     - static_configs:
          - target:
   # - alertmanager:9093
rule_files:
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "spring-actuator"
    metrics_path: '/actuator/prometheus'
    scrape_interval: 1s
    static_configs:
      - targets: ['localhost:8080']
```
* `job_name`: 수집하는 이름이다. 임의의 이름을 사용하면 된다.
* `metrics_path`: 수집할 경로를 지정한다.
* `scrape_interval`: 수집할 주기를 설정한다.
* `targets`: 수집할 서버의 IP, PORT를 지정한다.

### 프로메테우스 기본 기능
#### 기본 기능
* `Table` -> `Evaluation time`을 수정해서 과거 시간 조회 가능
* `Graph` -> 메트릭을 그래프로 조회 가능

#### 필터
레이블을 기준으로 필터를 사용할 수있다. 필터는 중괄호 `{ }` 문법을 사용한다.

##### 레이블 일치 연산자
* `=` 제공된 문자열과 정확히 동일한 레이블 선택
* `!=` 제공된 문자열과 같지 않은 레이블 선택
* `=~` 제공된 문자열과 정규식 일치하는 레이블 선택
* `!~` 제공된 문자열과 정규식 일치하지 않는 레이블 선택

예)
* `uri=/log`, `method=get` 조건으로 필터
  * `http_server_requests_seconds_count{uri="/log", method="get}`
* `/actuator/prometheus`는 제외한 조건으로 필터
  * `http_server_requests_seconds_count{uri!="/actuator/prometheus"}`
* `method`가 `GET`, `POST`인 경우를 포함해서 필터
  * `http_server_requests_seconds_count{method=~"GET|POST"}`
* `/actuator`로 시작하는 `uri`는 제외 조건으로 필터
  * `http_server_requests_seconds_count{uri!~"/actuator.*"}`

#### 연산자 쿼리와 함수
* `+` 덧셈
* `-` 뺄셈
* `*` 곱셈
* `/` 분할
* `%` 모듈로
* `^` 승수/지수

#### 게이지와 카운터
메트릭은 크게 보면 게이지와 카운터라는 2가지로 분류 할 수 있다.

#### 게이지(Gauge)
* 임의로 오르내일 수 있는 값
* 예) CPU 사용량, 메모리 사용량, 사용중인 커넥션

대표적인 게이지는 CPU 사용량(system_cpu_usage)이 있다.

#### 카운터(Counter)
* 단순하게 증가하는 단일 누적 값
* 예) HTTP 요청 수, 로그 발생 수

카운터는 계속 누적해서 증가하는 값이다. 따라서 계속 증가하는 그래프만 보게 될 것이다.
이렇게 증가만 하는 그래프에서 특정 시간에 얼마나 고객의 요청이 들어왔는지 한눈에 확인하기 매우 어렵다.
이 문제를 해결하기 위해 `increase()`, `rate()` 같은 함수를 지원한다.

#### increase
`increase()`를 사용하면 이런 문제를 해결할 수 있다. 지정한 시간 단위별로 증가를 확인할 수 있다.
마지막 `[시간]`을 사용해서 범위 벡터를 선택해야 한다.
예) `increase(http_server_requests_seconds_count{uri="/log"}[1m])`


#### rate()
범위 벡터에서 초당 평균 증가율을 계산한다.
`increase()`가 숫자를 직접 카운트 한다면 `rate()`는 여기에서 초당 평균을 나누어서 계산한다.
`rate(data[1m])`에서 `[1m]`이라고 하면 60초가 기준이 되므로 60을 나눈 수이다.
너무 복잡하게 생각하기 보다는 초당 얼마나 증가하는지 나타내는 지표로 보면 된다.

#### irate()
`rate()`와 유사한데, 범위 벡터에서 초당 순간 증가율을 계산한다. 급격하게 증가한 내용을 확인하기 좋다.

## 그라파나
### 연동
그라파나는 프로메테우스를 통해서 데이터를 조회하고 보여주는 역할을 한다.
그라파나에서 프로메테우스를 데이터소스로 사용해서 데이터를 읽어와야 한다.

* 설정(Configuration) -> DataSource
* `Add data source`를 선택한다.
* Prometheus를 선택한다.
* URL: `http://localhost:9090`(프로메테우스 URL)

### 대시보드
1. 대시보드 생성
2. 패널 생성

직접 대시보드를 입력할 수 도 있지만 그라파나는 이미 만들어둔 대시보드를 가져다가 사용할 수 있는 기능을 제공한다.

### 대시보드 - 공유 대시보드 활용
`http://grafana.com/grafana/dashboards` 는 수 많은 대시보드가 공개되어있다. 

### 메트릭을 통한 문제 확인
* CPU 사용량 초과
* JVM 메모리 사용량 초과
* 커넥션 풀 고갈
* 에러 로그 급증

---
# 모니터링 메트릭 활용
CPU 사용량, 메모리 사용량, 톰캣 쓰레드, DB 커넥션 풀과 같은 공통으로 사용되는 기술 메트릭은 이미 등록되어있다.
이런 이미 등록된 메트릭을 사용해서 대시보드를 구성하고 모니터링 하면된다.
여기에 더 나아가 비즈니스에 특화된 메트릭을 사용해서 대시보드를 구성하고 모니터링 하면 된다.
비즈니스에 관한 부분은 각 비즈니스 마다 구현이 다르다. 따라서 비즈니스 메트릭은 직접 등록하고 확인해야 한다.
여기서는 우리 비즈니스의 실시간 주문수, 취소수 또는 실시간 재고 수량을 메트릭을 등록하고 확인해보자.

## 메트릭 등록 - 카운터
### MeterRegistry
마이크로미터 기능을 제공하는 핵심 컴포넌트   
스프링을 통해서 주입받아서 사용하고, 이곳을 통해서 카운터, 게이지 등을 등록한다.

OrderServiceV1
```java
@Slf4j
public class OrderServiceV1 implements OrderService {
    private final MeterRegistry registry;
    private AtomicInteger stock = new AtomicInteger(100);
    
    public OrderServiceV1(MeterRegistry registry) {
        this.registry = registry;
    }
  
    @Override
    public void order() {
        log.info("주문");
        stock.decrementAndGet();
        
        Counter.builder("my.order")
              .tag("class", this.getClass().getName())
              .tag("method", "order")
              .description("order")
              .register(registry).increment();
    }
    
    @Override
    public void cancel() {
        log.info("취소");
        stock.incrementAndGet();
        Counter.builder("my.order")
              .tag("class", this.getClass().getName())
              .tag("method", "cancel")
              .description("order")
              .register(registry).increment();
    }
  
    @Override
    public AtomicInteger getStock() {
        return stock;
    }
}
```
* `Counter.builder(name)` 통해서 카운터를 생성한다. `name`에는 메트릭 이름을 지정한다.
* `tag`를 사용했는데, 프로메테우스에서 필터할 수 있는 레이블로 사용된다.
* 주문과 취소는 메트릭 이름은 같고 `tag`를 통해서 구분하도록 했다.
* `register(registry)`: 만든 카운터를 `MeterRegistry`에 등록한다. 이렇게 등록해야 실제 동작한다.

## @Counted
앞서 만든 `OrderServiceV1`의 가장 큰 단점은 메트릭을 관리하는 로직이 핵심 비즈니스 개발 로직에 침투했다는 점이다.
이런 부분을 AOP를 사용하면 분리할 수 있다.
마이크로미터는 이런 상황에 맞추어 AOP 구성요소를 이미 다 만들어 두었다.


```java
public class OrderServiceV2 implements OrderService {

    private AtomicInteger stock = new AtomicInteger(100);

    @Counted("my.order")
    @Override
    public void order() {
        log.info("주문");
        stock.decrementAndGet();


    }

    @Counted("my.order")
    @Override
    public void cancel() {
        log.info("취소");
        stock.decrementAndGet();

    }

    @Override
    public AtomicInteger getStock() {
        return stock;
    }
}
```
* `@Counted` 애노테이션을 측정을 원하는 메서드에 적용한다. 
* 그리고 메트릭 이름을 지정하면 된다.
* `tag`에 `method`를 기준으로 분류해서 적용한다.


```java
@Configuration
public class OrderConfigV2 {

    @Bean
    OrderService orderService() {
        return new OrderServiceV2();
    }

    @Bean
    public CountedAspect countedAspect(MeterRegistry registry) {
        return new CountedAspect(registry);
    }
}
```
* `@CountedAspect`를 등록하면 `@Counted`를 인지해서 `Counter`를 사용하는 AOP를 적용한다.
* 주의: `CountedAspect`를 빈으로 등록하지 않으면 `Counted` 관련 AOP가 동작하지 않는다.

## Timer
Timer는 좀 특별한 메트릭 측정 도구인데, 시간을 측정하는데 사용된다.
* 카운터와 유사한데, `Timer`를 사요하면 실행 시간도 함께 측정할 수 있다.
* `Timer`는 다음과 같은 내용을 한번에 측정해준다.
  * `seconds_count`: 누적 실행 수 - `카운터`
  * `seconds_sum`: 실행 시간의 함 - `sum`
  * `seconds_max`: 최대 실행 시간(가장 오래걸린 실행 시간) - `게이지`
  * 내부에 타임 윈도우라는 개념이 있어서 1~3분 마다 최대 실행 시간이 다시 계산된다.


OrderServiceV3
```java
public class OrderServiceV3 implements OrderService {

    private final MeterRegistry registry;
    private AtomicInteger stock = new AtomicInteger(100);

    public OrderServiceV3(MeterRegistry registry) {
        this.registry = registry;
    }


    @Override
    public void order() {
        Timer timer = Timer.builder("my.order")
                .tag("class", this.getClass().getName())
                .tag("method", "order")
                .register(registry);

        timer.record(() -> {
            log.info("주문");
            stock.decrementAndGet();
            sleep(500);
        });
    }

    @Override
    public void cancel() {
        Timer timer = Timer.builder("my.order")
                .tag("class", this.getClass().getName())
                .tag("method", "cancel")
                .register(registry);

        timer.record(() -> {
            log.info("취소");
            stock.decrementAndGet();
            sleep(500);
        });
    }

    @Override
    public AtomicInteger getStock() {
        return stock;
    }

    private static void sleep(int millis) {
        try {
            Thread.sleep(millis + new Random().nextInt(200));
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}

```
* `Timer.builder(name)` 통해서 타이머를 생성한다. `name`에는 메트릭 이름을 지정한다.
* `tag`를 사용했는데, 프로메테우스에서 필터할 수 있는 레이블로 사용된다.
* 주문과 취소 메트릭 이름은 같고 `tag`를 통해서 구별 하도록 했다.
* `register(registry)`: 만든 타이머를 `MeterRegistry`에 등록한다.
* 타이머를 사용할 때는 `timer.record()`를 사용하면 된다. 그 안에서 시간을 측정할 내용을 함수로 포함하면된다.


### @Timed
타이머는 `@Timed`라는 애노테이션을 통해 AOP를 적용할 수 있다.

## Gauge

```java
@Configuration
public class StockConfigV1 {

    @Bean
    public MyStockMetric myStockMetric(OrderService orderService, MeterRegistry meterRegistry) {
        return new MyStockMetric(orderService, meterRegistry);
    }

    @Slf4j
    static class MyStockMetric {
        private OrderService orderService;
        private MeterRegistry registry;

        public MyStockMetric(OrderService orderService, MeterRegistry registry) {
            this.orderService = orderService;
            this.registry = registry;
        }

        @PostConstruct
        public void init() {
            Gauge.builder("my.stock", orderService, service -> {
                log.info("stock gauge");
                return orderService.getStock().get();
            }).register(registry);
        }
    }
    
}
```
`my.stock`이라는 이름으로 게이지를 등록했다.
게이지를 만들 때 함수를 전달 했는데, 이 함수는 외부에서 메트릭을 확인할 때마다 호출된다.
이 함수의 반환값이 게이지의 값이다.

# 실무 모니터링 환경 구성 팁
모니터링 3단계
* 대시보드
* 애플리케이션 추적 - 핀포인트
* 로그

## 대시 보드
전체를 한 눈에 볼 수 있는 가장 높은 뷰

### 제품
마이크로미터, 프로메테우스, 그라파나

### 모니터링 대상
시스템 메트릭(CPU, 메모리)
애플리케이션 메트릭(톰캣 쓰레드 풀, DB 커넥션 풀, 애플리케이션 호출 수)
비즈니스 메트릭(주문수, 취소수)

## 애플리케이션 추적
주로 각각의 HTTP 요청을 추적, 일부는 마이크로서비스 환경에서 분산 추적

### 제품
핀포인트(오픈소스), 스카우트(오픈소스), 와탭(상용), 제니퍼(상용)

## 로그
가장 자세한 추적, 원하는데로 커스텀 가능
같은 HTTP 요청을 묶어서 확인할 수 있는 방법이 중요, MDC 적용

파일로 직접 로그를 남기는 경우     
일반 로그와 에러로그는 파일을 구분해서 남기자
에러 로그만 확인해서 문제를 바로 정리할 수 있음

클라우드에 로그를 저장하는 경우     
검색이 잘 되도록 구분

## 모니터링 정리
각각 용도가 다르다
관찰을 할 때는 전체 -> 점점 좁게
핀포인트는 정말 좋다. 강추 마이크로 서비스 분산 모니터링도 가능, 대용량 트래픽에 대응

알람
모니터링 툴에서 일정 이상 수치가 넘어가면, 슬랙, 문자 등을 연동

알람은 2가지 종류로 꼭 구분해서 관리
경고, 심각
경고는 하루 1번 정도 사람이 적접 확인해도 되는 수준(사람이 들어가서 확인)
심각은 즉시 확인해야 함. 슬랙 알림(앱을 통해 알림을 받도록), 문자, 전화


