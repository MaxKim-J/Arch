# 12_게시 구독 패턴

단방향 메시지 패턴이자 분산된 옵저버 패턴

## 설명

- 일련의 구독자가 특정 카테고리의 메시지를 수신하기 위해 구독을 등록
- 반면 게시자는 관련 구독자에게 배포되는 메시지를 생성
- 특기할 점은 게시자가 메시지의 수신자가 누구인지 미리 알 필요가 없음. 특정 메시지를 받기 위해서는 구독자가 자신의 관심사를 등록해야 하므로 게시자는 알 수 없는 수의 수신자와 함께 작업할 수 있음(이거는 Vue의 이벤트 에밋과 비슷한 내용인듯 함)
- 즉 게시 구독 패턴의 양쪽이 느슨하게 결합되어 있어 분산 시스템 노드르 통합하는데 이상적임
- 중간에 브로커를 하나 두면 구독자가 메시지의 게시자인 노드를 전-혀 알지 못해 브로커와만 상호작용하기 때문에 시스템 노드간의 분리가 더욱 개선됨. 

## 예제 - 채팅 어플리케이션

분산 아키텍처를 통합하는데 도움이 됨. 소켓을 이용한 채팅 어플리케이션 예제

### 서버

이렇게 기본적인 서버를 만들 수 있는데, 만약에 서버 인스턴스가 여러개라면 어떤 서버로 전송한 메시지는 그 서버에 연결되어있는 클라이언트들에게만 전파된다는 단점이 생긴다.

```js
const webSocketServer = require('ws').Server;

const server = require('http').createServer(
  // HTTP 서버를 만들고 정적 파일을 제공하기 위한 미들웨어
  require('ecstatic')({root: `${__dirname}/www`})
)

// 웹소켓 서버의 새 인스턴스
const wss = new WebSocketServer({server:server});

// 연결 이벤트에 대한 이벤트 리스너 첨부 => 연결대기 
wss.on('connection', ws => {
  console.log('client connected');
  // 새 메시지가 들어오면 연결된 모든 사용자에게 전파한다
  ws.on('message', msg => {
    console.log(`Message: ${msg}`);
    broadcast(msg);
  });
});

function broadcast(msg) {
  wss.clients.forEach(client => {
    client.send(msg);
  })
}
server.listen(process.argv[2] || 8080);
```

### 클라이언트

웹소켓 객체를 사용하여 Node 서버에 대한 연결을 초기화한 다음 수신을 시작하여 도착한 새로운 메시지는 div 엘리멘트에 표시된다.

```html
<html>
  <head>
  <script>
    var ws = new WebSocket('ws://' + window.document. local.host);
    ws.onmessage = function(message) {
      var msgDiv = document.createElement('div');
      msgDiv.innerHTML = message.data;
      document.getElementById('message').appendChild(msgDiv);
    };

    function sendMessage() {
      var message = document.getElementById('msgBox').value;
      ws.send(message);
    }
  </script>
  </head>
  <body>
    Messages:
    <div id='messages'></div>
    <input type='text' placeholder='send a message' id='msgBox'>
    <input type='button' onclick='sendMessage()' value='send'>
  </body>
</html>

```

### 메시지 브로커 추가(redis)

redis로 채팅 서버를 통합해보자. 클라이언트에서 보낸 메시지는 redis라는 매게체를 통해 다른 다수의 채팅 서버로 다시 이동하고, 연결된 모든 클라이언트에 전파된다. 여기서 게시 구독 로직을 이용한다.

```js
const webSocketServer = require('ws').Server;
const redis = require('redis');
const redisSub = redis.createClient();
const redisPub = redis.createClient();

const server = require('http').createServer(
  require('ecstatic')({root:`${__dirname}/www`})
)

const wss = new WebSocketServer({server:server});

// 연결 이벤트에 대한 이벤트 리스너 첨부 => 연결대기
wss.on('connection', ws => {
  console.log('client connected');
  // 새 메시지가 들어오면 연결된 모든 사용자에게 전파한다
  ws.on('message', msg => {
    console.log(`Message: ${msg}`);
    // 해당 채널에 메시지를 게시
    // 게시하면 하단의 구독 로직에서 send함
    redisPub.publish('chat_message', msg);
  });
});

// 서버에서는 해당 채널을 구독
redisSub.subscribe('chat_messages');

// 구독하고 있다가 메시지가 들어오면 client send
redisSub.on('message', (channel, msg) => {
  wss.clients.forEach(client => {
    client.send(msg);
  })
})
```

## reference

- [Node.js 디자인패턴 - 게시구독 패턴](http://www.yes24.com/Product/Goods/65050060)
