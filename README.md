### 部署步骤
1.选择合适的ES版本修改`.env`，建议采用默认7.1.1即可

2.修改合适的JVM堆内存，修改`docker-compose.yml`

3.`docker-compose up -d`启动容器组

4.启动ELK Stack后需要在logstash中安装json_lines插件(重要)
```
# 进入logstash容器
docker exec -it logstash /bin/bash
# 进入bin目录
cd /bin/
# 安装插件
logstash-plugin install logstash-codec-json_lines
# 退出容器
exit
# 重启logstash服务
docker restart logstash
```
作者在安装插件的过程中多次安装失败，由于网络原因，多尝试几次即可。

5.Spring Boot 集成`logstash-logback-encoder`即可。

### Spring Boot 整合 ELK

#### 添加依赖
```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>5.3</version>
</dependency>
```

#### logback-spring.xml
基础配置如下，需扩展自行配置即可，记得修改localhost
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 控制台 appender -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] %-5level - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <!--可以访问的logstash日志收集端口-->
        <destination>localhost:5000</destination>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="LOGSTASH"/>
    </root>
</configuration>
```

补充:
elasticsearch 安装ik插件
1. `docker exec -it elasticsearch /bin/bash`
2. `./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.8.0/elasticsearch-analysis-ik-7.8.0.zip`
3. 重启即可

