# 의존성 주입(Dependency Injection)

2020.08.03

SOLID 원칙에는 `의존성 역전`이라는게 있었고 **의존성 주입**은 이를 실현하기 위한 매커니즘이다.

## 이해

가장 단순하게 요약 : 함수나 객체에서 값을 인자로 주냐 안에서 가져오냐 차이

### 의존성 주입 안 한 객체

안에서 막 다 가져온다

```js
const Attendee = function(attendeeId) {
  this.attendee = attendeeId;
  this.service = new Service();
  this.messanger = new Messanger();
}

Attendee.prototype.reserve = function() {
  ...
}
```

### 의존성 주입 한 객체

인자로 받아서 멤버변수화한다

```js
// 멤버변수들은 인자에 의존한다!
const Attendee = function(service, messanger, attendeeId) {
  this.attendee = attendeeId;
  this.service = service;
  this.messanger = messanger;
}
```

## 장점

- 테스트하는 경우 의존 인자들을 다른곳에서 테스트하기 때문에 **이미 테스트가 이루어진** 객체들을 통해 믿음직한 코드를 유지한다
- 값을 안에서 조달하는 객체는 그 함수 본연의 기능만 테스트할 수 없게 된다. 그 값들에 대한 테스트도 진행해야하는데 그러면 **하나만 잘해야 하는** 테스트 코드가 애매해진다.
- 인자를 밖에서부터 분리해놓기 때문에 코드 재사용을 적극적으로 유도한다
- 객체간의 결합도를 줄여준다. 객체 안에서 값을 조달한다면 그 값을 객체에서 사용할 수 있을 정도로 처리하는 과정이 필요할 수도 있다. 하지만 객체 밖에서 이미 그 상태로 만들어져 온다면 객체에서 일련의 과정을 분리할 수 있다.


## 도입 고려해야 하는 지점

- 직접 인스턴스화를 피하고 인자로 주입하자는 거임. 꼭 new이야기가 아니라 어떤 값에 대하여 파라미터로 받아서 진행할 것인지 vs 함수 안에서 참조해서 진행할 것인지
- 객체 내부에서 발생할지 모를 에러를 테스트에서 고려해야 하나 혹은 따로 검증해야 하나 => 대부분의 경우는 따로 검증하는게 낫다
- 외부 자원(DB, HTTP 등)에 의존하고 있다면 분리시키는게 낫다.

## 라이브러리들

### 직접 구현해보기

#### 1. 세팅

```js
class DI {
  constructor() {
   this.registrations = [];
  }
  // Di.prototype.register()
   register(name, deps, func) {
    this.registrations[name] = {deps, func}
  }
}
```

사용할 함수들을 모두 등록할 수 있도록 registrations 배열을 만들어 초기화한다. 함수를 등록하는 메소드 register도 만든다. 

- deps : 의존성 목록을 저장하는 배열
- func : 함수 본체
- 이때 func는 함수 본체를 반환하는 함수 : 등록할 함수를 불러올 때 의존 객체를 매개변수로 넘겨줘서 온전한 함수가 되어야 함

#### 2. 주입

dep1, dep2는 의존성이 없고 main은 **이미 등록한 dep1, dep2에 의존**하는 함수다.

```js
const di = new Di();

di.register('dep1', [], function() {
  return function() {
    return 1;
  };
});

di.register('dep2', [], function() {
  return function() {
    return 2;
  };
});

// 이미 네임스페이스에 존재하는 함수에 의존하는 함수를 만들고 싶음
di.register('main', ['dep1', 'dep2'], function(dep1, dep2) {
  return function() {
    // 새로 만들어진 함수이지만, 의존성 주입으로써 class 내부의 기존 정보(함수)를 사용할 수 있다
    // 메서드 정의해서 사용하는거랑 무슨 차이지? 왜 일케하지?
    return dep1() + dep2();
  }
});
```

#### 3. 사용

registration.func에는 main 함수의 본체를 담은 성크가 있고 apply 함수로 deps를 매개변수로 넘겨준다. main 함수 본체에서는 의존성 객체 목록을 매개변수로 받아서 사용할 수 있는 것이다.

```js

class Di {
  get(name) {
    // name에 해당하는 함수 찾기
    const registration = this.registrations[name];
    const deps = [];
    if (registration === undefined) { return undefined; }

    // 함수의 의존성 배열을 순회하며 의존하는 함수를 registrations에서 찾아 새 배열에 넣는다
    registration.deps.forEach(depName => {
      deps.push(this.get(depName))
    });

    // name에 해당하는 함수에다가 의존성 배열을 인자로 삼아 apply(바로호출)
    // 첫번째 인자 skip(this는 그대로 유지)
    return registration.func.apply(undefined, deps);
    // function() {function() { return 1; }} 을 apply하면 function() { return 1; }
  }
}

const main = di.get('main');
main();

// main의 dept 배열 마지막 결과물 => main함수 본체에는 인자로 들어간다
[
  function() { return 1; },
  function() { return 2; }
]

// main 함수는 마지막으로 이렇게
function(dep1, dep2) {
  return function() {
    return dep1() + dep2();
  }
}
```

### typeDI 라이브러리

![의존성 주입](https://miro.medium.com/max/1022/1*mAhlq4o4MsqFwWp0bRjK4Q.png)

- 타입스크립트의 의존성 주입 라이브러리
- container을 선언할 수 있고 거기에 객체의 네임스페이스를 만들어 쑤셔넣을 수 있다.
- 인스턴스화가 필요한 클래스에 Service 데코레이터를 붙여주고, 필요한 의존성은 클래스 내부에 선언해주면 typedi가 알아서 의존성을 주입합니다.
- 우리는 직접 의존성의 인스턴스를 생성하지 않아도 됩니다. typedi에게 의존성 정보를 알려주고(constructor에 선언) 필요할 때 요구(Container.get)하면 됩니다.
- 즉, 우리가 아니라 typedi가 제어권을 갖는 주체로 동작합니다. 이를 제어권의 역전이라고 부릅니다.
- **new를 통해 새로운 인스턴스 만들어 넣는 수고를 덜어주는 역할**

## reference

- [김정환 블로그 - 의존성 주입](http://jeonghwan-kim.github.io/js/2017/02/17/dependency-injection.html)
- [TypeScript와 typedi로 의존성 주입 이해하기](https://medium.com/@HoseungJang/typescript%EC%99%80-typedi%EB%A1%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-5d83ef1977f9)
- [자바스크립트 패턴과 테스트 - 1.1.3 소프트웨어 공학 원칙을 적용하라](http://www.yes24.com/Product/Goods/33211518)