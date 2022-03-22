## RabbitMQ Exchange


1. Direct Exchange：消息根据routing key被发送到队列。routing key是一种消息属性。消息到达交换机，根据binding key到指定的队列里。

2. Topic Exchange：topic exchange根据wildcard匹配routing key和routing pattern，将消息发送到一个或者多个队列。

3. Fanout Exchange：无论routing key，将消息发送到所有绑定队列

4. Header Exchange：header exchange根据包括header和optional values的arguments route消息。


```
import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class DirectRabbitConfig {
    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange("directExchange", true, false);

    }

    @Bean
    public Queue directQueue() {
        return new Queue("directQueue", true);

    }

    @Bean
    Binding bindingDirect() {
        return BindingBuilder.bind(directQueue()).to(directExchange()).with("directRouting");
    }
}
```

```
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class RabbitSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @GetMapping("/sendMessage")
    public String sendMessage(){
        rabbitTemplate.convertAndSend("directExchange", "directRouting", "发送消息");
        return "ok";
    }
}

```

```
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = "directQueue")
public class RabbitConsumer
{
    @RabbitHandler
    public void process(String message){
        System.out.println("接收消息："+ message);
    }
}

```
