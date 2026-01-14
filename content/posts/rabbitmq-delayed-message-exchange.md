+++
title = 'RabbitMQ 实现延迟消息'
date = 2022-07-15T16:02:35+08:00
categories = ['中间件']
tags = ['mq', 'rabbitmq', '中间件']
summary = 'RabbitMQ 实现延迟消息，rabbitmq_delayed_message_exchange 插件。'
+++

## 下载延时消息插件

* 插件首页：[https://www.rabbitmq.com/community-plugins.html](https://www.rabbitmq.com/community-plugins.html)
* Github：[https://github.com/rabbitmq/rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)
* Github Releases：[https://github.com/rabbitmq/rabbitmq-routing-node-stamp/releases](https://github.com/rabbitmq/rabbitmq-routing-node-stamp/releases)

## 在 Docker 环境下，安装延迟消息插件

进入容器找到 plugins 目录

```bash
docker exec -it rabbitmq bash
##可以看到，plugins 就是存放 mq 插件的地方了
ls 
```

将插件复制到 plugins 目录下

```bash
cd /usr/etc/rabbitmq_plugins
docker cp rabbitmq_delayed_message_exchange-3.8.0.ez rabbitmq:/plugins
```

回到 plugins 目录，查看 plugins 中是否有 rabbitmq_delayed_message_exchange 插件

![202210111603258](https://xwls-oss.netlify.app/images/202210111603258.webp)

激活插件

```bash
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

![202210111604293](https://xwls-oss.netlify.app/images/202210111604293.webp)

重启 RabbitMQ

```bash
docker restart rabbitmq
```

进入 RabbitMQ 管理界面查看插件是否成功生效

![202210111604761](https://xwls-oss.netlify.app/images/202210111604761.webp)

OK, 完成以上工作，就可以编写 Java 代码发送延迟消息了。

## SpringBoot 中发送延迟消息

### Config

```java
package com.xjm.mid.compent.rabbitmq.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.CustomExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;

/**
 * @author jaymin
 * 2020/09/22
 */
@Configuration
public class RabbitMQDelayedMessageConfig {
    /**
     * 延迟消息交换机
     */
    public final static String DELAY_EXCHANGE = "jaymin.delay.exchange";
    /**
     * 队列
     */
    public final static String DELAY_QUEUE = "jaymin.delay.queue";
    /**
     * 路由 Key
     */
    public final static String DELAY_ROUTING_KEY = "jaymin.delay.routingKey";

    @Bean
    public CustomExchange delayMessageExchange() {
        Map<String, Object> args = new HashMap<>();
        args.put("x-delayed-type", "direct");
        //自定义交换机
        return new CustomExchange(DELAY_EXCHANGE, "x-delayed-message", false, false, args);
    }

    @Bean
    public Queue delayMessageQueue() {
        return new Queue(DELAY_QUEUE, false, false, false);
    }

    @Bean
    public Binding bindingDelayExchangeAndQueue() {
        return BindingBuilder.bind(delayMessageQueue()).to(delayMessageExchange()).with(DELAY_ROUTING_KEY).noargs();
    }
}
```

### Client

```java
package com.xjm.mid.compent.rabbitmq.web;

import com.xjm.mid.compent.rabbitmq.config.RabbitMQDelayedMessageConfig;
import com.xjm.mid.compent.rabbitmq.model.Letter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;

/**
 * @author jaymin
 * 2020/09/22
 */
@RestController
@RequestMapping("/message/delayed")
@Slf4j
public class DelayedMessageClient {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostMapping("/ten")
    public void sendDelayedMessage1(){
        Letter letter = new Letter();
        letter.setRecipient("福尔摩斯");
        letter.setContext("您的 10S 外卖到了！");
        Integer ttl = 10000;
        rabbitTemplate.convertAndSend(RabbitMQDelayedMessageConfig.DELAY_EXCHANGE, RabbitMQDelayedMessageConfig.DELAY_ROUTING_KEY, letter, message -> {
            // 设置过期时间
            message.getMessageProperties().setDelay(ttl);
            return message;
        });
        log.info("[发送时间] - [{}]-[过期时间]-[{}]", LocalDateTime.now(), ttl/1000);
    }

    @PostMapping("/five")
    public void sendDelayedMessage2(){
        Letter letter = new Letter();
        letter.setRecipient("福尔摩斯");
        letter.setContext("您的 5S 外卖到了！");
        Integer ttl = 5000;
        rabbitTemplate.convertAndSend(RabbitMQDelayedMessageConfig.DELAY_EXCHANGE, RabbitMQDelayedMessageConfig.DELAY_ROUTING_KEY, letter, message -> {
            // 设置过期时间
            message.getMessageProperties().setDelay(ttl);
            return message;
        });
        log.info("[发送时间] - [{}]-[过期时间]-[{}]", LocalDateTime.now(), ttl/1000);
    }
}
```

### Listener

```java
package com.xjm.mid.compent.rabbitmq.web;

import com.rabbitmq.client.Channel;
import com.xjm.mid.compent.rabbitmq.config.RabbitMQDelayedMessageConfig;
import com.xjm.mid.compent.rabbitmq.model.Letter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.time.LocalDateTime;

/**
 * @author jaymin
 * 2020/09/22
 */
@Component
@Slf4j
public class DelayMessageListener {

    @RabbitListener(queues = {RabbitMQDelayedMessageConfig.DELAY_QUEUE})
    @RabbitHandler
    public void receiveMessage(Channel channel, Message message, Letter letter) {
        log.info("[listenerDelayQueue 监听的消息] - [消费时间] - [{}] - [{}]", LocalDateTime.now(), letter.toString());
    }
}
```

### Result

![202210111605613](https://xwls-oss.netlify.app/images/202210111605613.webp)

rabbitmq 的延时插件极限时间是 8byte 长度 ms，大概 49 天。如果你的延时时间很长，建议配合定时任务进行处理。

`message.getMessageProperties().setDelay(ttl);` 这种方式设置延迟时间话，理论上最多 24 天左右。因为 `setDelay()` 的参数是 `Integer` 类型的，`25` 天的时候就超过了 `Integer` 的长度了，变成了负数，下游立马收到消息的。建议改为 `message.getMessageProperties().getHeaders().put("x-delay",delay);`