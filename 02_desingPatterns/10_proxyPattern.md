# 10_프록시 패턴

실제 기능을 수행하는 객체 대신 가상의 객체를 사용해 로직의 흐름을 제어하는 디자인 패턴. 원래 하려던 기능을 수행하며 그외의 부가적인 작업을 수행하거나, 비용이 많이 드는 연산을 실제로 필요한 시점에 수행할 수 있음. 종단 관심사랑 연관이 되어있는듯?

## Node에서의 적용

- 다른 객체에 대한 접근을 제어하는 객체. 다른 객체를 subject라고 하고 프록시와 대상은 동일한 인터페이스를 가지고 있고 이를 통해 다른 인터페이스와 완전히 호환되로록 바꿀 수 있음(surrogate)
- 프록시는 대상에서 실행될 작업의 전부 또는 일부를 가로채서 해당 동작을 향상시키거나 보완함
- 프록시는 각 작업을 대상으로 전달하여 추가적인 전처리 또는 후처리로 동작을 향상시키는 역할을 함

## 구현

순수함수로 만들거냐 아니면 객체를 직접 수정할거냐

### 오브젝트 컴포지션

기능을 확장하거나 사용하기 위해 객체가 다른 객체와 결합되는 기술

```js
// object composition
// 프로토타입 체인을 무결성있게 관리하기 위해서 프로토타입을 통해 메소드를 수정
// proxy instance of subject == true


function createProxy(subject) {
  // 지정된 객체의 프로토타입 속성을 반환한다.
  const proto = Object.getPrototypeOf(subject);

  // 생성자 함수
  function Proxy(subject) {
    this.subject = subject;
  }

  // 지정된 프로포토타입 객체 및 속성을 갖는 새 객체를 만든다.(create)
  // Proxy 생성자 프로토타입을 치환해서 subject의 instance를 만들도록 하는 과정
  Proxy.prototype = Object.create(proto);

  // 프록시된 메소드에서 대상 객체의 메소드 기능을 확장시킨다
  Proxy.prototype.hello = function() {
    return this.subject.hello() + 'world!';
  }

  // 나머지는 단순히 대상에 위임하기
  Proxy.prototype.goodbye = function() {
    // this delegation후 바로 호출(그냥 그대로 호출)
    return this.subject.goodbye.apply(this.subject, arguments)
  }

  // 요거는 이제 팩토리
  return new Proxy(subject)
}
module.exports = createProxy;

// 간단하게는 요런식으로
// 객체 리터럴 + 팩토리

function createProxy(subject) {
  return {
    hello:() => (subject.hello() + 'world'),
    goodBye: () => (subject.goodbye.apply(subject, arguments))
  };
}
```

### 객체 증강(멍키 패칭 - 메소드 빌림)

객체의 개별 메소드를 프록시하는 가장 실용적인 방법일 수 있음. 메소드를 프록시된 구현체로 대체하여 직접 대상을 수정하는 것으로 이루어짐

```js
function createProxy(subject) {
  // 메소드를 분리 => 이경우에는 메소드를 직접 수정하게 됨
  const helloOrig = subject.hello;
  // 분리한 메소드를 호출된 맥락의 this에 바인딩해서 호출
  subject.hello = () => (helloOrig.call(this) + ' world!');
  return subject;
}
```

- 컴포지션은 프록시를 만드는 가장 안전한 방법으로 간주될 수 있음. 대상을 그대로 두어서 원래의 동작은 변경하지 않는 순수함수를 통해 객체를 확장시키기 때무네
- 유일한 단점을 모든 메소드를 수동으로 위임해야된다는 것
- 객체증강은 대상을 수정하므로 위임과 관련된 여러가지 불편함이 없음. 대상을 수정하는 것이 큰 문제가 되지 않는다면 모든 상황에서 선호됨

## 적용

- 함수 후킹이라고도 한다. 낚아채기! AOP를 구현하는 한 방법이기도 하다.
- 대개 개발자가 특정 메소드들 전후에 실행 훅을 설정할 수 있도록 하는 방식으로 AOP가 구현된다. jest의 beforeEach같은거, react의 render 전에 들어가는 로직들이 다 훅이다.
- 미들웨어와 거의 사실 유사하다. 미들웨어 패턴에서 발생하는 것과 같이 어떤 함수의 입력/출력 전처리 후처리를 할 수 있기 때문에...(후킹을 한다는 점에서) 때로는 미들웨어와 유사한 파이프라인을 사용하여 동일한 메소드에 대해 여러 훅을 등록할 수도 있을 것이다.

## ES2015 PROXY

ES2015에서 나온 문법. 이건 정말 쓸데가 있지 않을까??

```js
// target : 프록시가 적용되는 객체
// handler : 프록시의 동작을 정의하는 특수한 객체
const proxy = new Proxy(target, handler);

const scientist = {
  name: 'nikola',
  surname: 'tesla'
}

// 핸들러 객체에서는 해당 작업이
// 프록시 인스턴스에서 수행될 때 자동으로 호출되는 트랩 메소드라는 사전에 정의된 이름을 가진
// 선택적 메소드들이 포함되어 있음

// 약간 getter같기도 함. 가공해서 내놓기!
const uppercaseScientist = new Proxy(scientist, {
  get: (target, property) => target[property].toUpperCase()
})

console.log(uppercaseScientist.name, uppercaseScientist.surname)
```

- target으로 지정한 객체 내부 일반 속성에 대한 접근을 가로챌 수 있다
- API가 단순히 프록시 객체의 생성을 용이하게하는 단순한 래퍼가 아니기 때문에 가능. 개발자가 객체에서 수행할 수 있는 많은 작업을 가로채서 사용자 정의화 할 수 있게 됨.

```js
// 모든 짝수를 포함하는 가상의 배열을 만듬
// 빈 배열을 타겟으로 사용하여 핸들러에 get과 has라는 트랩을 정의한다.
const evenNumbers = new Proxy([], {
  // 배열 요소에 대한 접근을 가로채 주어진 인덱스에 해당하는 짝수를 반환
  get:(target, index) => index * 2,
  // has트랩은 in 연산자의 사용을 가로채 주어진 숫자가 짝수인지 여부를 검사
  has:(target, number) => number % 2 === 0
});
```

## reference

- [Node.js 디자인패턴 - 프록시 패턴](http://www.yes24.com/Product/Goods/65050060)