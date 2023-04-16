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
> 콘솔 로그의 수준을 변경하는 방법은 application.yml 과 logback-spring.xml 에서 설정하는 방법이 있다. 
> application.yml 은 설정하는 난이도가 비교적 쉽지만, 실제 제품에 사용하기엔 한계가 있고 세부적인 설정이 불편하기 때문에 logback-spring.xml 로 관리하는 편이 더 좋다고 생각한다.
> 
> `appender`: 콘솔 혹은 파일 중 어디로 출력할 지 지정 및 콘솔에 출력되는 형식을 지정한다.  
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
>   <file>./logs/info.log</file> 
>   <filter class="ch.qos.logback.classic.filter.LevelFilter">
>       <level>INFO</level>
>       <onMatch>ACCEPT</onMatch> 
>       <onMismatch>DENY</onMismatch> 
>   </filter> 
>   <encoder>
>       <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS Z, Asia/Seoul}] [%thread] %-5level %logger{35} - %msg%n</pattern>
>   </encoder>
>   <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
>       <fileNamePattern>./was-logs/info.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern> 
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
> ex) `<fileNamePattern>./info.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>` 인 경우 파일로 나눠지는 단위가 하루이므로 maxHistory 의 단위도 하루가 된다.  
> ex2) `<fileNamePattern>./info.%d{yyyy-MM}.%i.log.gz</fileNamePattern>` 인 경우 파일로 나눠지는 단위가 한달이므로 maxHistory 의 단위도 한달이 된다.  
> 
> logger: log를 남길 대상들을 의미한다. logger와 appender의 조합으로 특정 classpath는 콘솔에 로그를 남기고, 
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

### 참조사이트
> [Logback 으로 쉽고 편리하게 로그 관리를 해볼까요? ⚙️](https://tecoble.techcourse.co.kr/post/2021-08-07-logback-tutorial/)

---

## Log4j2
### TODO
> TODO

---

## Log 라이브러리 비교
### 비교
> TODO

---

## Log for Amazon S3
### TODO
> TODO

---

## Log for Amazon CloudWatch
### TODO
> TODO
