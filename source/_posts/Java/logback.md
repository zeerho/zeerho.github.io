```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <contextName>ExampleApp</contextName>

  <conversionRule conversionWord="exampleMsg" converterClass="com.example.ExampleMsgConverter" />

  <!-- 输出到控制台 -->
  <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
    <encoder charset="UTF-8">
      <pattern>
        %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %cn %-5level %logger{36}#%method %msg%n
      </pattern>
    </encoder>
  </appender>

  <appender name="dispatchLogAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>${applog_path:-.}/dispatch.%d{yyyy-MM-dd}.log</fileNamePattern>
      <maxHistory>7</maxHistory>
    </rollingPolicy>
    <encoder>
      <pattern>%d{MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>

  <if condition='property("spring.profiles.active").contains("development")'>
    <then>
      <root level="INFO">
        <appender-ref ref="stdout" />
      </root>
    </then>
  </if>
  <if condition='property("spring.profiles.active").contains("functional") || property("spring.profiles.active").contains("test")'>
    <then>
      <root level="INFO">
        <appender-ref ref="stdout" />
      </root>
    </then>
  </if>
  <if condition='property("spring.profiles.active").contains("production")'>
    <then>
      <root level="INFO">
        <appender-ref ref="stdout" />
      </root>
    </then>
  </if>
</configuration>
```
