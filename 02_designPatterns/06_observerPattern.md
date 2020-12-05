# 06_옵저버 패턴

## 설명

- 관찰자 패턴은 Node.js의 반응적인 특성을 모델링하고 콜백을 완벽하게 보완하는 이상적인 해결책
- 관찰자 패턴은 상태 변화가 일어날 때 관찰자에게 알릴 수 있는 객체를 정의하는 것
- 콜백 패턴과의 가장 큰 차이점은 Subject가 실제로 여러 관찰자들에게 알릴 수 있다는 점임. 전통적인 연속 전달 스타일 콜백은
일반적으로 그 결과를 하나의 Listner인 콜백에만 전파함
- 어떤 코드에서 흥미로운 일이 생겼을 때 누가 받는지 상관없이 일단 알림을 보낼 수 있다는 거
- Vue에서 많이 본 그 패턴이기도 하

## EventEmitter 클래스

- 관찰자 패턴은 node의 코어에 내장되어 있고, EventEmitter 클래스를 통해 사용할 수 있음
- 이 클래스를 사용하여 특정 유형의 이벤트가 발생되면 호출될 하나 이상의 함수를 Listner로 등록할 수 있음
- EventEmitter은 프로토타입이며 코어 모듈로부터 익스포트 

```js
const EventEmitter = require('events').EventEmitter;
const eeInstance = new EventEmitter();
```

- 필수 메소드
    - on(event, listner) : 주어진 이벤트 유형에 대해 새로운 listner등록
    - once: 이 메소드는 새 이벤트를 생성하고 listner에게 전달할 추가적인 인자 지원
    - removeListner : 이 메소드는 지정된 이벤트 유형에 대한 listner을 제거

## 예제

```js
const EventEmitter = require('events').EventEmitter;
const fs = require('fs');

// 에미터 에밋을 리턴하는 함수(발생자)
// emitter를 리턴하는 것 => 이벤트를 발생시키는 것
function findPattern(files, regex) {
  const emitter = new EventEmitter();
  files.forEach(function(File) {
    fs.readFile(file, 'utf8', (err, content) => {
    // 파일을 읽는동안 오류가 났을때
    if (err) return emitter.emit('error', err);
    // fileread: 이벤트를 읽을 때 발생하는 이벤트
    emitter.emit('fileread', file);
    let match;
    if (match = content.match(regex)) {
      match.forEach(elem => emitter.emit('found', file, elem));
    }
 });
});
  return emitter
}

// 에밋을 받는 함수(관찰자)
findPatter(
  ['flleA.txt', 'fileB.json'], 
  /hello \w+/g
)
.on('fileRead', file => console.log(file+'was read'))
.on('found', (file, match) => console.log('Matched'+file+match))
.on('error', err => console.log(err.message))
```

- 이벤트 에미터는 이벤트가 비동기적으로 발생한 경우 이벤트 루프에서 손실될 수 있기 때문에 콜백에서와 같이 예외가 발생해도 예외를 바로 throw할 수 없다. 대신 error이벤트를 발생시키고 error 객체를 전달한다.
- 일반적인 이벤트 에미팅 말고도 다른 기능을 가지고 있는 일반적인 객체에서 어떠한 동작을 캐치하기 위해서 주로 사용한다. 일반적인 객체에다가 코드를 끼워넣어서 관찰 가능하게 만드는데 의미가 있는 것.

## 주요 사항

```js
function helloEvents() {
  const eventEmitter = new EventEmitter();
  setTimeOut(() => eventEmitter.emit('hello', 'hello world'), 100);
  return eventEmitter
}

function helloCabllback(callback) {
  setTimeOut(() => callback('hello world'), 100);
}
```

- 이벤트는 콜백과 마찬가지로 동기식, 또는 비동기식으로 생성될 수 있음
- 차이점 : 리스너를 등록할 수 있는 방법. 이벤트가 비동기적으로 발생한다면(뒤로 미뤄지는 것이니 - emit)EventEmitter가 초기화된 후에도 프로그램은 새로운 리스너를 등록할 수 있음. 이벤트가 이벤트 루프 다음 사이클이 될 때까지는 실행되지 않음.
- 결과 자체가 비동기 방식으로 **결과가 반환**되어야 하는 경우에는 콜백을 사용한다. 대신 이벤트는 일어난 무엇인가를 **전달**할 필요가 있을때 사용해야 한다. 
- 동일한 이벤트가 여러번 발생할 수도 있고 전혀 발생하지 않을수도 있다면, 반복적인 상황에 놓인다면 결과보다는 정보가 전달되어야 한다. 
- 리액트의 function props는 일종의 콜백, 뷰의 $event emit은 일종의 이벤트

## reference

- [Node.js 디자인패턴 - 관찰자 패턴](http://www.yes24.com/Product/Goods/65050060)