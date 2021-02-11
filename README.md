# sampleSpringIntegration
Sample of Spring Integration with HTTP/SOAP

##개요
Spring Integration은 Interface들을 위한 하나의 큰 틀이다.
각 Interface들의 Message(Data)간 전달을 위하여 큰 틀에서 추상화 시켰다.

##구조
###Message
모든 System은 결국 Data의 전달을 위한 짜임임.
대부분의 Data는 실질적인 Value와 META Data로 나뉨.
Java는 Type 기반의 언어이므로 Data의 Type이 변경 될 때마다 Interface를 수정해야한다면
손이 많이 가게 됨. 따라서 실질적인 Value(Payload)를 Generic(T)로, META Data를 Header로 사용함.

	public interface Message<T> {
		T getPayload();
		MessageHeaders getHeaders();
	}



####Message Header
Header는 Key-Value의 형태를 띄며 불변객체(Read-Only)이다.

	public final class MessageHeaders implements Map<String, Object>, Serializable {
        // …
    }

Header의 값은 아래와 같은 방법으로 불러온다.

	MessageHeaders messageHeaders = message.getHeaders();
	Object someValue = messageHeaders.get("someKey");
	CustomerId customerId = messageHeaders.get("customerId", CustomerId.class);
	Long timestamp = messageHeaders.getTimestamp();

####Message Builder
Message Interface에는 불변성을 깨뜨릴 수 있는 Setter가 존재하지 않는다.
따라서 긴 파라메터 목록을 가진 Constructor로 생성해야 하는데,
편의를 위하여 MessageBuilder Class가 Message 객체를 생성한다. (Method Chaining)

	Message<String> msg1 = MessageBuilder.withPayload("test")
                                        .setHeader("foo", "bar")
                                        .build();
    
    Message<String> msg2 = MessageBuilder.fromMessage(msg1)
                                        .build();

    assertEquals("test", msg2.getPayload());   // True
    assertEquals("bar", msg2.getHeaders()      // True
                            .get("foo"));


###Message Channel (Pipe)
Message가 전달되는 통로.
Message와 마찬가지로 통로의 경우도
Publisher와 Subscriber에 종속적이지 않게 설계되어있다.
따라서 간편한 확장이 가능하다.

[<img src="https://postfiles.pstatic.net/20140715_169/fltltmxjs_1405384897269NnMiF_JPEG/channel.jpg?type=w2">](https://blog.naver.com/fltltmxjs/220008414333)
[<img src="https://image.slidesharecdn.com/springintegration-150420040755-conversion-gate02/95/spring-integration-44-638.jpg?cb=1429517566">](https://www.slideshare.net/WangeunLee/spring-integration-47185594)
[<img src="https://image.slidesharecdn.com/springintegration-150420040755-conversion-gate02/95/spring-integration-45-638.jpg?cb=1429517566![img_3.png](img_3.png)">](https://www.slideshare.net/WangeunLee/spring-integration-47185594)


###Message Endpoint (Filter)
Message Channel을 통해 Message 송수신 등과 같은 처리를 돕는 모듈

####Message Endpoint의 종류
| Component | Description |
| :--------- | :----------- |
| Gateway |	Business Logic에서 Message 송수신을 쉽게 도움.<br/><br/>Message를 Channel별로 DI받아 생성하여 해당 Channel로 보냄<br/> → Coupling 발생 → Gateway 사용<br/> → 사전에 개발자가 정의한 interface로 Proxy Bean을 제공함으로써 Message 생성 / Channel로 전송하는 Messaging Framework에 종속된 개발 과정이 제거됨.<br/> → POJO를 직접 send 가능해짐. |
| Service Activator | Message가 입력 채널에 도착했을 때 특정 Bean의 Method를 호출 <br/><br/>Reply-Channel이 SubscribableChannel 계열인 경우 Gateway 사용 불가<br/> → Service Activator 사용 → Input-Channel에는 Gateway와 달리 PollableChannel / SubscribableChannel 계열 모두 사용 가능함 |
| Channel Adapter | 외부 System과 Message 송수신 |
| Transformer | Input-Channel로 들어온 Payload를 특정 목적에 맞게 변환하여 Output-Channel로 전달 |
| Filter | 특정 Channel로 어떤 Message는 전달하지 말아야 할지 Filtering |
| Router | Message를 한 개 이상의 Channel로 보냄 |
| Splitter | Channel로 들어온 Message를 여러 조각으로 나눔 |
| Aggregator | 여러 Message를 조합하여 하나의 Message로 통합 |
| Message Bridge | 다른 종류의 Messaging Channel이나 Adapter를 연결 |
| Message Enricher | 수신 Message에 추가적인 정보를 더하여 확장, 수정된 객체를 하위 Consumer에게 전송 |

참고
* https://docs.spring.io/spring-integration/docs/current/reference/html/index.html
* https://blog.naver.com/fltltmxjs/220008414333
* https://www.slideshare.net/WangeunLee/spring-integration-47185594
