# logback
logback+Slf4j

## 动态设置日志
``` java
import ch.qos.logback.classic.Logger;
import ch.qos.logback.classic.Logger;
import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.classic.encoder.PatternLayoutEncoder;
import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.rolling.RollingFileAppender;
import ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy;
import ch.qos.logback.core.util.FileSize;

import org.slf4j.LoggerFactory;

LoggerContext context = (LoggerContext) LoggerFactory.getILoggerFactory();
Logger logger = loggerContext.getLogger("ROOT");

// 构建Appender
RollingFileAppender<ILoggingEvent> appender = new RollingFileAppender<>();
appender.setContext(context); // 必须
appender.setFile("path/to/your/name.log");

// 构建RollingPolicy
SizeAndTimeBaseRollingPolicy rollingPolicy = new SizeAndTimeBaseRollingPolicy();
rollingPolicy.setFileNamePattern("path/to/your/name.%d{yyyy-MM-dd}.%i.log");
rollingPolicy.setParent(appender);
rollingPolicy.setContext(context); // 必须
rollingPolicy.setMaxHistory(7);
rollingPolicy.setMaxFileSize(FileSize.valueOf("20MB"));
rollingPolicy.setTotalSizeCap(FileSize.valueOf("2000MB");
rollingPolicy.start(); // 必须

appender.setRollingPolicy(rollingPolicy);
appender.setAppend(true);

// 构建Encoder
PatternLayoutEncoder encoder = new PatternLayoutEncoder();
encoder.setCharset(StandardCharsets.UTF_8);
encoder.setPattern(%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] [%t] %logger{50}.%M:%L - %m%n);
encoder.setContext(context); // 必须
encoder.start(); // 必须

appender.setEncoder(encoder);
appender.setName("name");
appender.start(); // 必须

logger.addAppender(appender);
logger.setLevel(Level.valueOf("info"));
logger.setAddtive(false);
```
