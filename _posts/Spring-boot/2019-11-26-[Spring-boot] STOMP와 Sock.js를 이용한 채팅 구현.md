---
title: STOMP와 Sock.js를 이용한 채팅 구현
description: STOMP와 Sock.js를 이용한 채팅 구현
categories:
 - Spring-boot
tags:
---  


# 채팅 구현 순서 정리

Spring boot 기반 앱   
채팅내용과 채팅방 정보를 모두 DB에 저장하는걸로 구현했다.

​

## 1) 의존성 추가

build.grade
```java
//websocket
    compile('org.springframework.boot:spring-boot-starter-mustache')
	compile('org.springframework.boot:spring-boot-starter-websocket')
	compile('org.webjars.bower:jquery:3.3.1')
	compile('org.webjars:sockjs-client:1.1.2')
	compile('org.webjars:webjars-locator:0.30')
	compile('org.webjars:stomp-websocket:2.3.3')
	compile("org.webjars:webjars-locator-core")
//webjar
	compile 'org.webjars:jquery:3.2.1'
	compile 'org.webjars:jquery-ui:1.11.4'
	compile 'org.webjars:bootstrap:3.3.7'

```
이게 다 필요하진 않을텐데 여러예제를 참고하다보니 많아졌다.. ;-;

​

​

## 2) 구독&발행 설정

WebSocketConfig.java
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
	@Override
	public void configureMessageBroker(MessageBrokerRegistry registry) {
		registry.enableSimpleBroker("/topic"); //메시지 브로커 등록
		registry.setApplicationDestinationPrefixes("/app"); //메시지발행. 도착경로에 대한 prefix
	}

	@Override
	public void registerStompEndpoints(StompEndpointRegistry registry) {
		registry.addEndpoint("/websocket").withSockJS(); //stomp websocket의 연결 endpoint
	}
}

```
## 3) 채팅 메시지, 채팅방 Model 생성

ChatRoom.java
```Java
@Getter
@Setter
public class ChatRoom {
	private int idChatRoom;
    private String chatRoom_name;

    @OneToOne
	@JoinColumn(name = "user_key")
	private String user_key;
}

```

ChatMessage.java
```Java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class ChatMessage {
	// 메시지 타입 : 입장, 채팅, 퇴장
    public enum MessageType {
        ENTER, TALK, QUIT
    }
    private MessageType chatMessage_type; // 메시지 타입
    private String chatMessage_writer; // 메시지 보낸사람
    private String chatMessage_message; // 메시지

    @OneToOne
   	@JoinColumn(name = "idChatRoom")
    private int idChatRoom; // 방번호 외래키

}

```
## 4) 메시지 발행/저장

ChatMessageController.java
```Java
@Controller
public class ChatMessageController { // publisher 구현
	@Autowired
	ChatMessageMapper chatMessageMapper;

	private final SimpMessagingTemplate template;
	// SimpMessagingTemplate @EnableWebSocketMessageBroker를 통해서 등록되는 bean이다.
	// 특정 Broker로 메시지를 전달한다.

	@Autowired
	public ChatMessageController(SimpMessagingTemplate template) {
		this.template = template;
	}

	@MessageMapping("/chat")  //메시지 처리
	public void chat(ChatMessage chatMessage) {
		int id = chatMessageMapper.selectCount();
		chatMessageMapper.insertChatMessage(id+1,chatMessage.getChatMessage_type().toString(),
				chatMessage.getChatMessage_writer(), chatMessage.getChatMessage_message(),
				chatMessage.getIdChatRoom());
		template.convertAndSend("/topic/chat/"+chatMessage.getIdChatRoom() , chatMessage); //발행
	}

}

```
## 5) 채팅방 생성/조회

ChatRoomController.java
```java
@RequiredArgsConstructor
@Controller
@RequestMapping("/chat")
public class ChatRoomController {
	   @Autowired
	   ChatRoomMapper chatRoomMapper;

	    // 모든 채팅방 목록 반환
	    @RequestMapping(path = "/rooms", method = RequestMethod.GET)
	    public @ResponseBody List<ChatRoom> room(@RequestParam(value = "user_key") String user_key) {
	 	  return chatRoomMapper.findAllRoomByUserKey(user_key);
	    }

	    // 채팅방 생성
	    @RequestMapping(path = "/room", method = RequestMethod.POST)
	    public @ResponseBody void createRoom(@RequestParam String name, @RequestParam String user_key) {
	    	int id = chatRoomMapper.countChatRoom();
	        chatRoomMapper.insertChatRoom(id+1,name,user_key);
	    }

	    // 특정 채팅방 조회
	    @RequestMapping(path = "/room/{roomId}", method = RequestMethod.GET)
	    public @ResponseBody ChatRoom roomInfo(@PathVariable int roomId) {
	        return chatRoomMapper.findRoomById(roomId);
	    }
}
```
​

## 6) 채팅 TEST 뷰 만들기

app.js
```Javascript
var stompClient = null;
function setConnected(connected) {
    $("#connect").prop("disabled", connected);
    $("#disconnect").prop("disabled", !connected);
    if (connected) {
        $("#conversation").show();
    }
    else {
        $("#conversation").hide();
    }
    $("#greetings").html("");
}

function connect() {
    var socket = new SockJS('/websocket');
    stompClient = Stomp.over(socket);
    stompClient.connect({}, function (frame) {
        setConnected(true);
        console.log('Connected: ' + frame);
        stompClient.subscribe('/topic/greetings', function (greeting) {
            showGreeting(JSON.parse(greeting.body).content);
        });
        stompClient.subscribe('/topic/chat', function (chat) { //구독
        	showChat(JSON.parse(chat.body));
        });
    });
}

function disconnect() {
    if (stompClient !== null) {
        stompClient.disconnect();
    }
    setConnected(false);
    console.log("Disconnected");
}

function sendName() {
    stompClient.send("/app/hello", {}, JSON.stringify({'chatMessage_writer': $("#chatMessage_writer").val()}));
}

function sendChat() { //Message객체에 writer, message내용, roomId 전달
	stompClient.send("/app/chat", {}, JSON.stringify({'chatMessage_writer': $("#chatMessage_writer").val(),
		'chatMessage_message': $("#chatMessage_message").val(), 'idChatRoom':roomId }));
}
//stringify 문자열화 시키는거

function showGreeting(message) {
    $("#greetings").append("<tr><td>" + message + "</td></tr>");
}
function showChat(chat) { //채팅내역 출력
    $("#greetings").append("<tr><td>" + chat.chatMessage_writer + " : " + chat.chatMessage_message + "</td></tr>");
}
$(function () {
    $("form").on('submit', function (e) {
        e.preventDefault();
    });
    $( "#connect" ).click(function() { connect(); });
    $( "#disconnect" ).click(function() { disconnect(); });
    $( "#send" ).click(function() { sendName(); });
    $( "#chatSend" ).click(function(){ sendChat(); });
});
```
**채팅방별로 메시지를 전송하기 위해 필요한 과정**

app.js에서 구독할때 /topic/chat/{roomid} 를 구독하면

거기서 발행한 message의 idChatroom에 값이 담기고

ChatMessageController의 chat메서드가 실행되서 거기에서

template.convertAndSend("/topic/chat/"+chatMessage.getIdChatRoom() , chatMessage);

이렇게 보내면 해당 채팅룸으로 메세지가 전송됨

​

index.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello WebSocket</title>
    <link href="/webjars/bootstrap/css/bootstrap.min.css" rel="stylesheet">
    <link href="/main.css" rel="stylesheet">
    <script src="/webjars/jquery/3.2.1/jquery.min.js"></script>
    <script src="/webjars/sockjs-client/sockjs.min.js"></script>
    <script src="/webjars/stomp-websocket/stomp.min.js"></script>
    <script src="/app.js"></script>
</head>
<body>
<noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
    enabled. Please enable
    Javascript and reload this page!</h2></noscript>
<div id="main-content" class="container">
    <div class="row">
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="connect">WebSocket connection:</label>
                    <button id="connect" class="btn btn-default" type="submit">Connect</button>
                    <button id="disconnect" class="btn btn-default" type="submit" disabled="disabled">Disconnect
                    </button>
                </div>
            </form>
        </div>
        <div class="col-md-6">
            <form class="form-inline">
                <div class="form-group">
                    <label for="name">What is your name?</label>
                    <input type="text" id="chatMessage_writer" class="form-control" placeholder="Your name here..." />
                </div>
                <div class="form-group">
                	<label for="message">Input Message</label>
                	<input type="text" id="chatMessage_message" class="form-control" placeholder="message.." />
                </div>
                <button id="chatSend" class="btn btn-default" type="button">Chat Send</button>
            </form>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <table id="conversation" class="table table-striped">
                <thead>
                <tr>
                    <th>Messages</th>
                </tr>
                </thead>
                <tbody id="greetings">
                </tbody>
            </table>
        </div>
    </div>
</div>
</body>
</html>
```
## 7) Mybatis이용한 Mapper구현 (코드 생략)

​

​

### 배운점 :  

Stomp, websocket 개념, api 짜기

##### @PathVariable & @RequestParam 차이

@PathVariable : "/room/{roomId}" 이런 형태 url을 만들기 위해서 사용  

@RequestParam : “/room?name=“aa””

​

### 어려웠던 점 :

connection과 send가 잘 안되서 애를 많이 먹었다.

그리고 각 채팅방으로 메시지를 구분하여 보내기 위해서는 동적인 주소가 필요한데,

이는 내가 처음에 사용했던 @SendTo어노테이션으로 구현이 잘 안된다고해서 MessageTemplate를 이용하여 메시지 전송함

​

##### 메시지전송 메서드 구현 방법

1) @SendTo 어노테이션 사용
```Java
@MessageMapping("/chat")
@SendTo("/topic/chat")
public ChatMessage chat(ChatMessage chatMessage) throws Exception {
  return new ChatMessage(chatMessage.getRoomId(),chatMessage.getSender(),chatMessage.getMessage(),chatMessage.getType());
}
```
2) SimpMessagingTemplate 사용

```Java
private final SimpMessagingTemplate template;
	@Autowired
	public ChatMessageController(SimpMessagingTemplate template) {
		this.template = template;
	}
@MessageMapping("/chat")
	public void chat(ChatMessage chatMessage) {
		template.convertAndSend("/topic/chat", chatMessage);
}
```

##### 참고한 블로그 :
<https://daddyprogrammer.org/post/4691/spring-websocket-chatting-server-stomp-server/>

<https://hwiveloper.github.io/2019/01/10/spring-boot-stomp-websocket/>
