# spring-logging-starter

## 로깅 설계
### 로그 수준
>| 수준 | 개요 | 설명                                                                                            | 출력지         | 운용 시 대응      |
>|---|---|-----------------------------------------------------------------------------------------------|-------------|--------------|
>| FATAL	| 치명적인 에러 | 	프로그램의 비정상 종료를 수반하는 것.콘솔 등에 즉시 출력하는 것을 상정                                                     | 콘솔 파일       | 	즉각 대응이 필요   | 
>| ERROR |	에러 | 	예상하지 못한 심각한 문제가 발생하는 경우, 즉시 조취를 취해야 할 수준의 레벨                                                 | 	콘솔 파일      | 	영업 시간 내의 대응 |
>| WARN |	경고 | 	Deprecated 된 API 사용 및 API의 부적절한 사용 오류에 가까운 사상 등. 실행시에 생긴 이상이라고 할 수 없지만 정상과도 다른 뭔가 뜻하지 않은 문제	 | 콘솔 파일       | 	다음 릴리스까지 대응 |
>| INFO |	정보 | 	운영에 참고할만한 사항, 중요한 비즈니스 프로세스가 완료됨                                                             | 	콘솔 파일	     | 대응 필요        |
>| DEBUG |	디버깅 | 	정보 시스템 동작 상황에 대한 자세한 정보, 개발 단계에서 사용	                        | 파일| 	출력하지 않는다    |
>| TRACE |	트레이스 | 	정보 디버깅 정보보다 더욱 상세한 정보, 모든 레벨에 대한 로깅이 추적되므로 개발 단계에서 사용함         | 파일| 	출력하지 않는다    |
>
> 로그 레벨 설정
> TRACE > DEBUG > INFO > WARN > ERROR > FATAL   
> 로그 레벨을 TRACE 로 설정하면 TRACE, DEBUG, INFO, WARN, ERROR, FATAL 까지 모두 보여준다.  
> 로그 레벨을 ERROR 로 설정하면 ERROR, FATAL 만 보여준다.

### 참조사이트
> [로그 설계 지침](https://jacking75.github.io/ETC_log/)

---

## Logback
### Log4j 란?
> Log4j는 가장 오래된 프레임워크이며 Apache 의 Java 기반 Logging Framework 다. xml, properties 파일로 로깅 환경을 구성하고, 콘솔 및 파일 출력의 형태로 로깅을 할 수 있게 도와준다.
> 로그 레벨의 경우는 6단계로 구성되어있다.

### Logback 이란?
> Logback 은 log4j 이후에 출시된 Java 기반 Logging Framework 중 하나로 가장 널리 사용되고 있다. 
> SLF4j 의 구현체이며 Spring Boot 환경이라면 별도의 dependency 추가 없이 기본적으로 포함되어 있다.  
> 
> Logback 은 log4j 에 비해 향상된 필터링 정책, 기능, 로그 레벨 변경 등에 대해 서버를 재시작할 필요 없이 자동 리로딩을 지원한다는 장점이 있다.  

### SLF4J 란?
> SLF4J 는 JCL 의 가진 문제를 해결하기 위해 클래스 로더 대신에 컴파일 시점에서 구현체를 선택하도록 변경시키기 위해 도입된 것이다. 

### 로그 파일 작성
> 콘솔 로그의 수준을 변경하는 방법은 application.yml 과 `logback-spring.xml` 에서 설정하는 방법이 있다.   
> application.yml 은 설정하는 난이도가 비교적 쉽지만, 실제 제품에 사용하기엔 한계가 있고 세부적인 설정이 불편하기 때문에 logback-spring.xml 로 관리하는 편이 더 좋다고 생각한다.  
> 로그 설정 파일의 이름을 `logback-spring.xml` 로 사용시 SpringBoot 에서는 별다른 설정 없이 해당 파일을 설정파일로 사용한다.  
> 하지만 Profile 마다 로그 설정 파일을 다르게 하고 싶어서 로그 설정 파일을 여러개로 두는 경우에는 `application.yml` 에 별도의 설정이 필요하다.  
> 예를 들어 로그 설정 파일의 이름을 `logback-dev.xml` 로 지정한 경우 application.yml 에 다음 설정을 추가한다.
> ```yml
> ...
> logging:
>    config: classpath:logback-dev.xml
> ```

### property
> appender 작성 시 변수로 사용할 값을 설정한다. 
> 예시 파일에서는 일반 로그가 저장될 디렉터리 경로 및 이전 로그가 저장될 디렉터리 경로를 각각 property 로 설정하였다.  
> ```xml
> ...
> <property name="logsPath" value="./logs"/>
> <property name="wasLogsPath" value="./was-logs"/>
> <property name="layoutPattern"
>           value="[%d{yyyy-MM-dd HH:mm:ss.SSS Z,Asia/Seoul}] [%thread] %-5level %logger{36} - %msg%n"/>
> <property name="maxFileSize" value="10MB"/>
> <property name="maxHistory" value="180"/>
> ...
> ```

### appender
> 콘솔 혹은 파일 중 어디로 출력할 지 지정 및 콘솔에 출력되는 형식을 지정한다.    
> 콘솔 출력 appender 예시
> ```xml
> <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
>   <encoder>
>      <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS Z, Asia/Seoul}][%thread] %-5level %logger{36} - %msg%n</pattern>
>   </encoder>
> </appender>
> ```
> name: appender 의 name 속성은 사용자가 임의로 지정할 수 있는 속성이다. 여기서는 편의를 위해서 STDOUT 으로 지정하였다.  
> class: 콘솔에 출력할 appender 의 경우 `ch.qos.logback.core.ConsoleAppender` 를 고정값으로 넣는다.  
> 날짜 패턴 `%d{yyyy-MM-dd HH:mm:ss.SSS Z, Asia/Seoul}`: Z 는 타임 존을 표시한다. `, Asia/Seoul`은 표시 날짜의 타임 존을 지정한다.  
> 
> 파일 출력 appender 예시  
> ```xml
> <appender name="INFO_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
>   <file>${LOG_DIR}/info.log</file> 
>   <filter class="ch.qos.logback.classic.filter.LevelFilter">
>       <level>INFO</level>
>       <onMatch>ACCEPT</onMatch> 
>       <onMismatch>DENY</onMismatch> 
>   </filter> 
>   <encoder>
>       <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS Z, Asia/Seoul}] [%thread] %-5level %logger{35} - %msg%n</pattern>
>   </encoder>
>   <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
>       <fileNamePattern>${WAS_LOG_DIR}/info.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern> 
>       <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
>           <maxFileSize>10MB</maxFileSize> <!-- 한 파일의 최대 용량 -->
>       </timeBasedFileNamingAndTriggeringPolicy>
>       <maxHistory>180</maxHistory> <!-- 한 파일의 최대 저장 기한(day) -->
>   </rollingPolicy>
> </appender>
> ```
> file: 파일을 저장할 경로를 지정한다.  
> 
> filter: onMatch: ACCEPT, onMismatch: DENY 시에는 로깅 레벨로 지정한 레벨 이외에는 이 appender 로 로깅하지 않는다는 것을 의미한다.  
> INFO 및 상위 레벨(WARN, ERROR, FATAL)을 모두 로깅하기 위해서는 onMatch: ACCEPT, onMismatch: ACCEPT 로 설정한다.  
> 필수 값은 아니며 레벨별 필터링이 필요없을 경우 <filter> 태그를 사용하지 않는다.  
> 
> encoder: 로깅 패턴을 지정한다.  
> 
> rollingPolicy: 파일로 로깅할 시 한 파일에는 하루의 로깅 양만 저장하는 것이 좋다.  
> 로깅 파일을 날짜 별로 나눠주는 역할을 rollingPolicy 로 지정한다. class 를 `TimeBasedRollingPolicy` 로 고정한다.    
> 
> timeBasedFileNamingAndTriggeringPolicy: 시간 기반으로 파일을 나눠주기 위한 설정이며, class 에 `SizeAndTimeBasedFNATP` 를 설정하여
> 시간 뿐만 아니라 하나의 로그 파일이 가질 수 있는 최대 용량도 지정하여 용량에 따라서도 파일을 나눠주도록 설정한다.  
> 
> maxFileSize: 한 파일당 최대 파일 용량을 저장한다. log 내용의 크기도 IO 성능에 영향을 미치기 때문에 되도록 너무 크지 않는 사이즈로 지정하는게 좋다.
> 10MB 권장한다.
> 
> maxHistory: maxHistory 가 30이고, Rolling 정책을 일 단위로 하면 30일 동안만 저장되고, 월 단위로 하면 30개월간 저장된다. 
> 예를들어 30일동안 30개의 파일이 유지됐다면 오래된 파일부터 삭제된다.  
> `fileNamePattern` 에서 로그 파일의 이름 뒤의 날짜 패턴에 따라서 maxHistory 의 값의 단위가 결정된다.  
> ex) `<fileNamePattern>${WAS_LOG_DIR}/info.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>` 인 경우 파일로 나눠지는 단위가 하루이므로 maxHistory 의 단위도 하루가 된다.  
> ex2) `<fileNamePattern>${WAS_LOG_DIR}/info.%d{yyyy-MM}.%i.log.gz</fileNamePattern>` 인 경우 파일로 나눠지는 단위가 한달이므로 maxHistory 의 단위도 한달이 된다.  

### logger
> log를 남길 대상들을 의미한다. logger와 appender의 조합으로 특정 classpath는 콘솔에 로그를 남기고, 
> 어떤 로그는 에러 발생시 이메일을 발송하고, 어떤 상황에서는 custom 한 로그 이벤트 처리를 할 수 있도록 다양한 처리를 가능하도록 구성할 수 있다.
>
> root: Root 는 최상단 logger로서 아무 설정도 안할 경우 root 의 log level에 따라 로그 이벤트를 남길지 안남길지 설정이 가능해진다.  
> 
> 예시)
> ```xml
> ...
> <root level="DEBUG"> <!-- 모든 logger 와 관계 없이 로그 레벨이 DEBUG 이면 아래 STDOUT appender 를 통해 로그를 출력한다. -->
>     <appender-ref ref="STDOUT"/>
> </root>
>
> <!-- 
> starter.spring.logging.runner 패키지 아래의 클래스에서 출력하는 로그들 중 로그 레벨이 INFO 인 경우에만 
> INFO_LOG appender 및 ERROR_LOG appender 를 통해 로그를 출력한다.
> -->
> <logger name="starter.spring.logging.runner" additivity="false">
>     <level value="INFO"/>
>     <appender-ref ref="INFO_LOG"/>
>     <appender-ref ref="ERROR_LOG"/>
> </logger>
> ```
> 
> root 와 logger 가 사용하는 appender 가 다른 경우라면 logback-spring.xml 을 통해 logger 를 따로 사용하는 것이 좋다.  
> 하지만 root 와 logger 가 사용하는 appender 가 같은 경우에는 logback-spring.xml 에는 root 만 두고 logger 는 application.yml 에 정의하는 것이 깔끔하다.  
> application.yml  
> ```yaml
> logging:
>   level:
>       web: debug
>       starter.spring.logging.runner: debug
> ```

### 참조사이트
> [Logback 으로 쉽고 편리하게 로그 관리를 해볼까요? ⚙️](https://tecoble.techcourse.co.kr/post/2021-08-07-logback-tutorial/)
> [Spring의 logging 구조와 logback에 대해 알아보자](https://velog.io/@gehwan96/logback-설정)

---

## Log4j2
### 개요
> log4j2는 Spring Boot에 기본으로 적용되어있는 logback 이후에 나온 라이브러리로 성능이 더 뛰어나다.  
> 멀티스레드 환경에서 Async Logger의 경우 Logback보다 처리량이 18배 더 높고 대기 시간이 훨씬 더 짧다.

### Dependency
> ```groovy
> ...
> configurations {
>     ...
>     all {
>         // logback 과의 충돌 방지
>         exclude module: 'spring-boot-starter-logging'
>     }
>     ...
> }
> ...
> dependencies {
>     ...
>     implementation 'org.springframework.boot:spring-boot-starter-log4j2'
> }
> ```

### 로그 파일 작성
> 콘솔 로그의 수준을 변경하는 방법은 application.yml 과 `log4j2.xml` 에서 설정하는 방법이 있다.
> application.yml 은 설정하는 난이도가 비교적 쉽지만, 실제 제품에 사용하기엔 한계가 있고 세부적인 설정이 불편하기 때문에 log4j2.xml 로 관리하는 편이 더 좋다고 생각한다.
> 로그 설정 파일의 이름을 `log4j2.xml` 로 사용시 SpringBoot 에서는 별다른 설정 없이 해당 파일을 설정파일로 사용한다.  
> 하지만 Profile 마다 로그 설정 파일을 다르게 하고 싶어서 로그 설정 파일을 여러개로 두는 경우에는 `application.yml` 에 별도의 설정이 필요하다.  
> 예를 들어 로그 설정 파일의 이름을 `log4j2-dev.xml` 로 지정한 경우 application.yml 에 다음 설정을 추가한다.
> ```yml
> ...
> logging:
>    config: classpath:log4j2-dev.xml
> ```

### Properties
> appender 작성 시 변수로 사용할 값을 설정한다.
> 예시 파일에서는 일반 로그가 저장될 디렉터리 경로 및 이전 로그가 저장될 디렉터리 경로를 각각 Property 로 설정하였다.
> ```xml
> ...
> <Proerties>
>   <Property name="logsPath" value="./logs"/>
>   <Property name="wasLogsPath" value="./was-logs"/>
>   <Property name="layoutPattern"
>             value="[%d{yyyy-MM-dd HH:mm:ss.SSS Z,Asia/Seoul}] [%thread] %-5level %logger{36} - %msg%n"/>
>   <Property name="maxFileSize" value="10MB"/>
>   <Property name="maxHistory" value="180"/>
> </Proerties>
> ...
> ```

### Appenders
> ConsoleAppender: 콘솔에 로그를 출력한다.  
> FileAppender: 파일에 로그를 저장한다.  
> RollingFileAppender: 파일에 로그를 저장하되, 로그 파일의 크기나 날짜 등을 기준으로 파일을 자동으로 분리한다.  
> SocketAppender: TCP/IP 소켓 통신을 통해 로그 이벤트를 원격지에 전달한다.  
> JmsAppender: JMS(Java Message Service)를 통해 로그 이벤트를 메시지로 전송한다.  
> KafkaAppender: Apache Kafka를 통해 로그 이벤트를 전송한다.  
> AsyncAppender: 로그 이벤트를 비동기 방식으로 처리하며, 다른 Appender를 포함할 수 있다.  
> 이 외에도 JDBCAppender, JPAAppender, CassandraAppender, MongoDBAppender 등이 있다.  
> 
> **Console Appender**  
> ```xml
> <Properties>
>   ...
>   <Property name="layoutPattern"
>         value="[%d{yyyy-MM-dd HH:mm:ss.SSS Z,Asia/Seoul}] [%thread] %-5level %logger{36} - %msg%n"/>
> </Properties>
> 
> <Appenders>
>    <Console name="STDOUT" target="SYSTEM_OUT">
>        <PatternLayout pattern="${layoutPattern}"/>
>    </Console>
>    ...
> </Appenders>
> ```
> Console: 콘솔 창에 로그를 출력하기 위한 Appender 설정 시 Console 태그를 이용한다.  
> name: appender 의 name 속성은 사용자가 임의로 지정할 수 있는 속성이다. 여기서는 편의를 위해서 STDOUT 으로 지정하였다.  
> target: 콘솔에 출력할 appender 의 경우 `SYSTEM_OUT` 를 고정값으로 넣는다.  
> 날짜 패턴 `%d{yyyy-MM-dd HH:mm:ss.SSS Z, Asia/Seoul}`: Z 는 타임 존을 표시한다. `, Asia/Seoul`은 표시 날짜의 타임 존을 지정한다.
>
> **RollingFile Appender** 
> ```xml
> <Properties>
>   <Property name="logsPath" value="./logs"/>
>   <Property name="wasLogPath" value="./was-logs"/>
>   <Property name="layoutPattern"
>             value="[%d{yyyy-MM-dd HH:mm:ss.SSS z}{Asia/Seoul}] [%thread] %-5level %logger{36} - %msg%n"/>
>   <Property name="maxFileSize" value="10MB"/>
>   <Property name="maxHistory" value="180"/>
> </Properties>
>  
> <Appenders>
>    ...
>    <RollingFile name="DEBUG_LOG">
>        <LevelRangeFilter minLevel="DEBUG" maxLevel="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
>        <FileName>${logsPath}/debug.log</FileName>
>        <FilePattern>${wasLogPath}/debug.%d{yyyy-MM-dd}.%i.log.gz</FilePattern>
>        <FileOwner>ec2-user</FileOwner>
>        <FileGroup>ec2-user</FileGroup>
>        <FilePermissions>rw-r--r--</FilePermissions>
>        <PatternLayout>
>            <Pattern>${layoutPattern}</Pattern>
>        </PatternLayout>
>        <Policies>
>            <SizeBasedTriggeringPolicy size="${maxFileSize}"/>
>            <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
>        </Policies>
>       <DefaultRolloverStrategy>
>           <Delete basePath="${wasLogPath}">
>               <IfFileName glob="debug.*.log.gz">
>                   <IfAccumulatedFileCount exceeds="${maxHistory}"/>
>               </IfFileName>
>           </Delete>
>       </DefaultRolloverStrategy>
>    </RollingFile>
>    
>    <Async name="ASYNC_DEBUG_LOG" includeLocation="true">
>       <AppenderRef ref="DEBUG_LOG"/>
>    </Async>
> </Appenders>
> ```
> RollingFile: 파일에 로그를 출력하기 위한 Appender 설정 시 RollingFile 태그를 이용한다.  
> name: Appender 의 이름을 지정한다.  
> 
> LevelRangeFilter: Appender 가 수용할 로그 레벨을 지정한다. 
> minLevel, maxLevel: 수용할 로그의 최소 레벨 및 최대 레벨을 지정한다. 주의할 점은 minLevel 과 maxLevel 을 
> 반대로 입력하면 Appender 를 통한 로그 기록이 제대로 동작 안한다. 
> 로그 레벨의 순서는 다음과 같다: FATAL(작음) < ERROR < WARN < INFO < DEBUG < TRACE(큼)  
> 예를 들어 minLevel="ERROR" maxLevel="DEBUG" 를 설정하면 로그가 정상적으로 기록되지만, 
> minLevel="DEBUG" maxLevel="ERROR" 로 설정하면 로그가 정상적으로 기록되지 않는다.  
>
> FileOwner, FileGroup, FilePermissions: 파일 권한 관련 옵션이다. 
> 생략해도 상관없으며 Production 환경에서 설정이 필요한 경우 사용하면 될듯하다.    
>
> PatternLayout: 로그 메시지 패턴을 지정한다.  
> 
> SizeBasedTrigggeringPolicy: 파일의 사이즈를 기준으로 파일을 나눈다. 예를 들어 size="10MB" 로 설정했다고 가정하면
> 로그 파일의 사이즈가 10MB 를 넘어가면 파일을 나눠서 다음 파일에 기록한다.  
> 
> TimeBasedTriggeringPolicy: 시간을 기준으로 파일을 나눈다. 여기서 interval 을 통해서 시간의 간격을 지정할 수 있는데
> interval 의 단위는 `FilePattern`에서 지정한 최소날짜(위 예제에서는 일(day))를 기준으로 한다.  
> 
> DefaultRolloverStrategy: 로그 파일을 가질 최대 갯수를 지정한다.  
> TODO
> 
> **Async Appender**  
> 로그를 기록을 메인 스레드에서 하는 것이 아니라 로그만 출력하는 스레드를 별도로 두어 로그를 출력하기 위한 Appender 이다.
> 
> includeLocation: Location 정보는 일반적으로 로그 이벤트가 발생한 코드의 파일 이름, 클래스 이름, 메서드 이름, 
> 라인 번호 등을 포함한다. 이 정보를 로그에 포함하면 디버깅이나 분석 시 로그 이벤트가 발생한 위치를 쉽게 파악할 수 있어서 유용하다.  
> 
> AppenderRef: Async Appender 는 기존에 만들었던 Appender 를 Async 로 동작하도록 해주는 Appender 임으로 
> ref 를 통해서 기존에 만들었던 Appender 를 참조하도록 한다.  

### Loggers
> log를 남길 대상들을 의미한다. logger와 appender의 조합으로 특정 classpath는 콘솔에 로그를 남기고,
> 어떤 로그는 에러 발생시 이메일을 발송하고, 어떤 상황에서는 custom 한 로그 이벤트 처리를 할 수 있도록 다양한 처리를 가능하도록 구성할 수 있다.
>
> ```xml
> ...
> <Loggers>
>     <Root level="INFO">
>         <AppenderRef ref="STDOUT"/>
>     </Root>
>     <Logger name="org.springframework.web" level="DEBUG" additivity="false">
>         <AppenderRef ref="ASYNC_DEBUG_LOG"/>
>         <AppenderRef ref="ASYNC_ERROR_LOG"/>
>     </Logger>
>     <Logger name="starter.spring.logging.runner" level="DEBUG" additivity="false">
>         <AppenderRef ref="ASYNC_DEBUG_LOG"/>
>         <AppenderRef ref="ASYNC_ERROR_LOG"/>
>     </Logger>
> </Loggers>
> ```
> Root: Root 는 최상단 logger로서 아무 설정도 안할 경우 root 의 log level에 따라 로그 이벤트를 남길지 안남길지 설정이 가능해진다.
> 
> Logger: Root 보다 우선하는 특정 패키지 내의 소스코드에서 적용되는 로깅 규칙을 지정한다.  
> Root 와 Logger 가 사용하는 Appender 가 다른 경우라면 log4j2.xml 을 통해 Logger 를 사용하는 것이 좋다.  
> 하지만 Root 와 Logger 가 사용하는 Appender 가 같은 경우에는 log4j2.xml 에는 Root 태그만 두고 Logger 태그는 제거 후 Logger 태그의 내용은 
> application.yml 에 정의하는 것이 깔끔하다.  
> application.yml
> ```yaml
> logging:
>   level:
>       web: debug
>       starter.spring.logging.runner: debug
> ```

### 참조사이트
> [[Spring Boot] 스프링 부트 로그 설정 (log4j2)](https://veneas.tistory.com/entry/Spring-Boot-스프링-부트-로그-설정-log4j2)  
> [Log4j 파일사이즈 및 파일갯수 세팅방법](https://woongnemonan.tistory.com/entry/Log4j-파일사이즈-및-파일갯수-세팅방법)  
> [Log4j2 LevelRangeFilter Example](https://howtodoinjava.com/log4j2/levelrangefilter-example/)  
> [Log4j2 AsyncLogger와 함께 하는 Logging 환경 구축](https://velog.io/@byeongju/Log4j2-AsyncLogger와-함께-하는-Logging-환경-구축)  
> [Log4J Async 처리하기](https://blog.naver.com/PostView.nhn?isHttpsRedirect=true&blogId=ccambo69&logNo=220195518406)  
> [Appenders](https://logging.apache.org/log4j/2.x/manual/appenders.html)  

---

## Log 라이브러리 비교
### Log4j2 와 Logback 비교
> **기능**  
> 두 라이브러리 모두 다양한 로그 수준을 지원하며, 로그를 파일이나 데이터베이스에 저장할 수 있다. 
> 또한 로그 출력 형식을 지정할 수 있다. 그러나 log4j2는 logback보다 더욱 세밀한 로깅 제어와 유연한 로그 분배 기능을 제공한다.
> 
> **설정 파일**  
> logback은 XML 또는 Groovy 기반의 설정 파일을 지원하며, log4j2는 XML, JSON, YAML 등 다양한 형식의 설정 파일을 지원한다.  
> 
> **성능**   
> log4j2는 멀티 스레드 환경에서 더욱 빠른 로그 처리 속도를 보인다. 또한 메모리 사용량도 logback보다 적다.  
> 
> **유연성**  
> log4j2는 로깅 시스템 자체가 모듈화되어 있어서, 원하는 로깅 구성 요소를 선택하여 구성할 수 있다. 
> 이를 통해 개발자는 필요한 로깅 기능만 선택하여 사용할 수 있다.
> 
> **지원**   
> logback은 Spring Framework의 공식 로깅 라이브러리로 사용되며, 많은 개발자들이 사용하고 있다. 
> 그러나 log4j2는 최근에 업데이트되어 많은 기능과 최신 기술을 지원한다.  
> 결론적으로, logback은 간단하고 쉽게 설정할 수 있으며, Spring Framework와의 연동성이 뛰어나다는 장점이 있다. 
> 반면에 log4j2는 logback보다 더욱 유연하고 성능이 우수하며, 다양한 설정 파일 형식과 모듈화된 로깅 시스템을 제공한다는 장점이 있다.  

---

## Log for Amazon S3
### TODO
> TODO

---

## Log for Amazon CloudWatch
### TODO
> TODO
