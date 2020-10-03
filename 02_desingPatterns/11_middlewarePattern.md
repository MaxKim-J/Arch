# 11_미들웨어 패턴

하위 서비스와 어플리케이션 사이의 접착체처럼 작용하는 모든 종류의 소프트웨어 계층

## node에서의 적용

- node의 가장 특징적인 패턴
- node에서는 express가 그 선구자
- 미들웨어 패턴은 개발자가 프레임워크 코어를 확장하지 않고도 현재 어플리케이션에 쉽게 추가할 수 있는 새 기능을 만들고 배포할 수 있는 효과적인 전략
- express 미들웨어의 작업 : 요청 본문의 구문 분석, 요청 및 응답 압축 및 해제, 액세스 로그 생성, 세션 관리, 암호화된 쿠기 관리, CSRF보호 제공
- 어플리케이션의 나머지 부분을 지원하고 실제 요청의 처리가 핵심 비즈니스 로직에만 집중할 수 있게 해주는 액세서리적 역할을 함
- 필터와 핸들러를 통해 시스템을 확장하기 위한 간결한 방법을 제공하려는 노력 - 의존성 역전의 원칙이 여기서도 적용됨
- 미들웨어는 비동기 순차 실행의 흐름으로 호출됨. 각 유닛은 이전 유닛의 실행 결과를 입력으로 받게 됨
- 각각의 미들웨어는 콜백을 호출하지 않거나 에러를 콜백에 전달함으로써 데이터 처리를 중단할 수 있음. 오류 상황은 대개 오류 처리 전용인 다른 일련의 미들웨어들을 실행시킴

## 예제 - 미들웨어 프레임워크 구축

네트워크를 통해 원자 메시지를 교환하기 위한 간단한 인터페이스를 제공하는 MQ를 사용해 분산 메시징 시스템 만들기

### 미들웨어 관리자

```js
module.exports = class ZMqMiddlewareManager {
  constructor(socket) {
    this.socket = socket;
    // 이 두 미들웨어 목록은 상호 보완적인 관계

    // 인바운드 : 요청에서 앱의 핵심까지
    this.inboundMiddleware = [];

    // 아웃바운드: 앱의 핵심에서 응답까지
    this.outboundMiddleware = [];
    socket.on('message', message => {
      this.executeMiddleware(this.inboundMiddleware, {
        data:message
      })
    })
  }

  // 메시지를 보내는 메소드, 메시지를 보내기 전에 미들웨어를 실행한다
  send(data) {
    const message = {
      data:data
    };

    this.executeMiddleware(this.outboundMiddleware, message, () => {
      this.socket.send(message.data);
    })
  }

  // 미들웨어를 등록하는 메소드
  // 파이프라인에 새로운 미들웨어 기능을 추가할 때 필요하며 쌍으로 제공
  use(middleware) {
    if (middleware.inbound) {
      this.inboundMiddleware.push(middleware.inbound);
    }
    if (middleware.outbound) {
      // 옼 이거뭐임? 배열 맨 앞에 삽입하는건가 신기쓰;;
      this.outboundMiddleware.unshift(middleware.outbound);
    }
  }

  // 미들웨어를 실행하는 함수
  // 실행할 미들웨어 배열, 보낼 메시지, 미들웨어 처리가 끝났을때 실행하는 콜백을 인자로 받는다
  // 비동기 순차 반복 패턴
  executeMiddleware(middleware, arg, finish) {
    function iterator(index) {
      // 미들웨어 끝까지 실행했을 때
      if (index === middleware.length) {
        return finish && finish();
      }
      // 인덱스 이터레이팅 하면서 특정 미들웨어를 실행하고 에러처리
      middleware[index].call(this, arg, err => {
        if(err) return console.log('에러가 낫다네' + err)
        // 여기서는 순회를 재귀로 구현했다
        iterator.call(this, ++index);
      })
    }
    // 진입점
    iterator.call(this, 0);
  }
}
```

### 미들웨어 만들기

```js
// 미들웨어 함수 쌍 생성
module.exports = () => {
  return {
    // 역직렬화
    inbound:function(message, next) {
      message.data = JSON.parse(message.data.toString());
      next();
    },
    // 직렬화
    outbound:function(message, next) {
      // 이진 데이터화
      message.data = new Buffer(JSON.stringify(message.data));
      next();
    }
  }
}
```

### 애플리케이션

```js
const zmqm = new ZmqMiddlewareManger(reply);

zmqm.use(jsonMiddleware.json())

zmqm.use({
  inbound:function (message, next) {
    console.log('receive' + message.data);
    if (message.data.action === 'ping') {
      this.send({action:'pong', echo:message.data.echo};)
    }
    next()
  }
})
```

## 예제 - koa의 제너레이터 미들웨어

제너레이터를 사용해 인바운드 아웃바운드의 미들웨어 경계를 간편하게 정의

### 등록

```js
const app = require('koa')();

app.use(function *() {
  this.body = {'now':new Date()};
})

app.listen(3000);
```

### 미들웨어 

next 사용에 주목하셈 

```js
const lastCall = new Map();

module.exports = function *(next) {
  // 인바운드 미들웨어
  const now = new Date();
  if (lastCall.has(this.ip) && now.getTime() - lastCall.get(this.ip).getTime() < 1000) {
    return this.status = 429;
  }

  // 위에 동기로직들 끝내고 next를 기다리게된다 이런 식이면, 그리고 아웃바운드로 넘어감
  // 제너레이터 함수와 yield를 사용해 미들웨어 실행이 일시 중지되어
  // 목록 내 다른 모든 미들웨어가 순차적으로 실행되고
  // 인바운드 미들웨어의 마지막 항목이 실행될 때만 아웃바운드 흐름이 시작될 수 이씀
  // 기발하다;; 
  yield next

  // 아웃바운드 미들웨어
  lastCall.set(this.ip, now);
  this.set('X-RateLimit-Reset', now.getTime() + 1000);

}
```

## reference

- [Node.js 디자인패턴 - 미들웨어 패턴](http://www.yes24.com/Product/Goods/65050060)
