# Redis 설정

- 2024.09.28: RedisSubscriber의 onMessage에서 LocalDateTime 타입을 역직렬화 할 때 자꾸 에러가 나서 트러블 슈팅 중이다.

- 역직렬화에서 에러가 터질 뿐이지, Redis에 채팅방 구독은 잘 된다.   

## RedisConfig

```java
@Configuration
@RequiredArgsConstructor
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String redisHost;

    @Value("${spring.data.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory(){
        // Redis 연동 설정 Host 주소와 Post를 주입해준 redisStandaloneConfiguration를
        RedisStandaloneConfiguration redisStandaloneConfiguration =
                new RedisStandaloneConfiguration("localhost", 6379);
        // LettuceConnectionFactory의 생성자로 다시 넣어준다.
        return new LettuceConnectionFactory(redisStandaloneConfiguration);
    }

//    @Bean
//    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory){
//        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
//        redisTemplate.setConnectionFactory(connectionFactory); //redisConnectionFactory()를 실행해서 연동 설정 함
//        // Key와 Value를 직렬화 해줄 Serializer를 주입해준다.
//        redisTemplate.setKeySerializer(new StringRedisSerializer());
//        redisTemplate.setValueSerializer(new StringRedisSerializer());
//
//        return redisTemplate;
//    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(String.class));
        return redisTemplate;
    }

    @Bean
    public RedisTemplate<String, PublishMessage> chatRedisTemplate(RedisConnectionFactory connectionFactory){
        RedisTemplate<String, PublishMessage> chatRedisTemplate = new RedisTemplate<>();
        chatRedisTemplate.setConnectionFactory(connectionFactory);
        chatRedisTemplate.setKeySerializer(new StringRedisSerializer());
        // Value 직렬화 과정에서 에러 발생할 수 있으니 try catch로 예외 처리
        chatRedisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(String.class));

        return chatRedisTemplate;
    }

    @Bean
    public ChannelTopic topic(){
        return new ChannelTopic("/chatRoom");
    }

    @Bean
    public RedisMessageListenerContainer redisMessageListener(RedisConnectionFactory connectionFactory,
                                                              MessageListenerAdapter listenerAdapter,
                                                              ChannelTopic channelTopic) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        // 이 설정은 listenerAdapter가 특정 채널(channelTopic)에서 발행된 메시지를 수신하도록 구성합니다.
        container.setConnectionFactory(connectionFactory);
        // 메시지 수신 준비 + 구독할 채널 설정
        // listenerAdapter가 특정 채널(channelTopic)에서 발행된 메시지를 수신하도록 구성합니다. 메시지 리스너 등록
//        container.addMessageListener(listenerAdapter, channelTopic);
        return container;
    }

    @Bean
    public ChannelTopic channelTopic() {
        return new ChannelTopic("chatroom");
    }
    // 메세지를 구독자에게 보내는 역할
    @Bean
    public MessageListenerAdapter listenerAdapter(RedisSubscriber redisSubscriber) {
        return new MessageListenerAdapter(redisSubscriber, "onMessage");
    }
}
```

## RedisPublisher

```java
@Service
@RequiredArgsConstructor
public class RedisPublisher {
    //RedisConfig에서 설정한 RedisTemplate과 타입을 맞춰야한다.
    private final RedisTemplate<String, Object> redisTemplate;
//    private final RedisTemplate<String, PublishMessage> redisTemplate; -> 주입이 안됨

    public void publish(ChannelTopic topic, Object obj){
        PublishMessage publishMessage = (PublishMessage) obj;
        redisTemplate.convertAndSend(topic.getTopic(), publishMessage);
    }

}

```


## RedisSubscriber

문제의 RedisSubscriber

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class RedisSubscriber implements MessageListener{

    private final RedisTemplate<String, Object> redisTemplate;
    private final ObjectMapper obejctMapper;
    private final SimpMessageSendingOperations messageTemplate;


    // Subscriber와 메시지 전달 로직 메서드 명을 MessageListenerAdapter()의 매개인자에 추가 -> RedisConfig 파일에서 설정
//    @Override
//    public void onMessage(String message) throws JsonProcessingException {
//        log.info("publish 전 message: {}", message);
//        PublishMessage publishMessage = obejctMapper.readValue(message, PublishMessage.class);
//        // PublishMessage를 구독자(sub한 사람들)에게 메시지를 보낸다.
//        messageTemplate.convertAndSend("/sub/chats" + publishMessage.getRoomId(), publishMessage);
//        log.info("publish 후 message: {}", publishMessage.getContent());
//
//
//    }

    @Override
    public void onMessage(Message message, byte[] pattern) {
        // 역 직렬화
        String message1 = redisTemplate.getStringSerializer().deserialize(message.getBody());

        log.info("publish 전 message: {}", message1);
        PublishMessage publishMessage = null;
        try {
//            obejctMapper.registerModule(new JavaTimeModule());
            publishMessage = obejctMapper.readValue(message1, PublishMessage.class);
            messageTemplate.convertAndSend("/sub/chats" + publishMessage.getRoomId(), publishMessage);
            // PublishMessage를 구독자(sub한 사람들)에게 메시지를 보낸다.
            log.info("publish 후 message: {}", publishMessage.getContent());
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }

//    @Bean
//    public ObjectMapper objectMapper() {
//        ObjectMapper objectMapper = new ObjectMapper();
//        // 자바 8 날짜/시간 모듈 추가
//        JavaTimeModule javaTimeModule = new JavaTimeModule();
//        // LocalDateTime 역직렬화에 대한 포맷 설정 (옵션)
//        javaTimeModule.addDeserializer(LocalDateTime.class,
//                new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
//        objectMapper.registerModule(javaTimeModule);
//        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
//
//        return objectMapper;
//    }
}

```

## PublishMessage

createAt을 주석처리해서 실행하면 잘 된다..

```java
package com.test.chat.entity;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import org.antlr.v4.runtime.misc.NotNull;

import java.io.Serializable;
import java.time.LocalDateTime;

@Getter
@NoArgsConstructor
@AllArgsConstructor
public class PublishMessage implements Serializable {

    /*
    * Redis에서 PublishMessage를 통해서 통신한다. -> Publisher or Subscriver
    * 따라서 Serializable 인터페이스를 상속한다.
    * */

    private static final long serialVersionUID = 2082503192322391880L;

    @NotNull
    private Long roomId;
    @NotNull
    private Long senderId;
    @NotNull
    private String content;

    // LocalDateTime 타입에 대한 직렬화, 역직렬화 도우미 선언
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    @JsonDeserialize(using = LocalDateDeserializer.class)
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createAt;
}

```
