---
dg-publish: false
---
## 1. 结合springboot
  
下面是一个使用 Spring Boot 和 RocketMQ 的消费消息的代码示例：

### 1. 在 pom.xml 文件中添加 RocketMQ 的依赖：
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>${rocketmq-spring-boot.version}</version>
</dependency>
```
### 2.配置nameserver
在 `application.properties` 或 `application.yml` 文件中设置 RocketMQ 的相关参数，例如 `rocketmq.name-server`：
```yaml
spring:
  rocketmq:
    name-server: localhost:9876

```
### 3. 生产消息
2.  创建一个生产者类，使用 `@RocketMQTemplate` 注解标记该类，并自动注入 `RocketMQTemplate` 对象：
```java
@Component
public class TestProducer {

    @Autowired
    private RocketMQTemplate rocketMQTemplate;

    public void sendMessage(String message) {
        rocketMQTemplate.convertAndSend("test-topic", message);
    }
}

```

### 4.消费消息
创建一个消费者类，使用 `@RocketMQMessageListener` 注解标记该类，并设置相关的参数，例如 `consumerGroup`、`topic` 和 `selectorExpression`：
```java
@Component
@RocketMQMessageListener(consumerGroup = "test-group", topic = "test-topic", selectorExpression = "*")
public class TestConsumer implements RocketMQListener<String> {

    @Override
    public void onMessage(String message) {
        System.out.println("Received message: " + message);
    }
}

```
在上面的示例中，`TestProducer` 类使用 `RocketMQTemplate` 对象来发送消息。`TestConsumer` 类实现了 `RocketMQListener<String>` 接口，并覆盖了 `onMessage` 方法。在该方法中，您可以编写处理消息的业务逻辑。 `@RocketMQMessageListener` 注解用于指定消费者组、主题和选择器表达式。

