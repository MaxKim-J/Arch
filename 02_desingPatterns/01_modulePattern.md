# 1. 모듈 패턴

import나 require을 사용하게끔 될 수 있는 기본 원리

## 설명

- 네임스페이스 : 수많은 함수 객체 변수들로 이루어진 코드가 전역 유효범위를 어지럽히지 않고, 애플리케이션이나 라이브러리를 위한 하나의 **representative한 전역 객체**를 만들고 모든 기능을 이 객체에 추가하는 것 => 모듈의 진입점
- 코드에 네임스페이스를 지정해주면 코드 내의 이름 충돌뿐만 아니라 이 코드와 같은 페이지에 존재하는 또다른 서드파티들과의 이름 충돌도 미연에 방지(의존성 해결)
- 모듈 : 전체 애플리케이션의 일부를 독립된 코드로 분리해서 만들어 놓은 것
- 독립된 모듈은 그 자체로 독자적, 그 안에서 모든 의존성을 해결해야 함 => 내부 변수 및 내부 함수를 모두 갖고 있다
- 데이터 감춤 + 임의로 함수를 호출하여 생성 or 선언과 동시에 실행

## 예시

### 기본적인 임의 모듈 패턴

```js
const MyApp = {}

MyApp.wildlifePreserve = function(animalMaker) {
  // 클로저, 은닉될 멤버
  const animals = [];

  return {
    addAnimal: function(species, sex) {
      animal.push(animalMaker.make(species,sex))
    },
    getAnimalCount:function() {
      return animal.length;
    }
  }
}

// 이렇게 사용 가능
const preserve = MyApp.wildlifePreserveSimulator(realAnimalMaker);
preserve.addAnimal(gorilla, female)
```

- 객체 리터럴 반환하기
- api를 반환할때 **클로저를 사용하는 꼴**이 된다. 함수의 실행 컨텍스트는 끝났지만 animals라는 변수는 api가 계속 접근할 수 있기 때문에 => 일종의 private 변수가 됨
- 이런 경우에는 인스턴스를 여러개 생성하여 사용할 수 있다(호출이 안됐으니까)
- 모듈을 만들 때 사실 클로저는 거의 필수인게, 모듈 객체의 메소드가 여러개이고 어떤 은닉 변수가 메소드들 사이에서 다양하게 사용될 가능성이 높기 때무네

### 즉시 실행(배포) 모듈

```ts
var module = (function () {
   // 은닉될 멤버 정의
  var privateKey = 0;
  function privateMethod() {
    return ++privateKey;
  }
  // 공개될 멤버 정의
  return {
    publickey : privateKey,
    publicMethod : function () {
      return privateMethod();
    }
  }
})(); //선언과 동시에 즉시 실행된다

console.log(module.publicMethod()); // 1
```

- 즉시 실행 모듈은 전역 실행 컨텍스트에서 즉시 실행하여 이름공간을 가진 전역변수에 할당된(배포된) 후 해당 모듈의 **싱글톤 인스턴스**가 된다.
- import문으로 선언하면 해당 모듈을 바로 사용할 수 있게 되는 것은 이 원리에 따른다.

### 모듈 생성의 원칙

1. 단일 책임 원칙을 잊지 말고 한 모듈에 한가지 일만 시키기 : 그래야 결속력이 강하고 다루기 쉬운, 아담한 API를 작성하게 된다
2. 모듈 자신이 쓸 객체가 필요하다면 의존성 주입 형태로 이 객체를 제공하는 방안을 고려하라(팩토리 주입 형태도)
3. 다른 객체 로직을 확장하는 모듈은 해당 로직의 의도가 바뀌지 않도록 분명히 밝혀라

## 기본 패턴 정의

```js
// 1. 네임스페이스를 설정하고 모듈을 정의 
var MyApp = {}

// 전역 객체
MyApp.modules = {}

// 2. 공개범위(특권메소드 등..)와 비공개 유효범위를 만든다 
MyApp.modules.libs = (function() {
  // 비공개 프로퍼티 API (public, previlege 멤버)
  return {

  };
}());

```

## reference

- [module pattern](https://webclub.tistory.com/5)
- [자바스크립트 패턴과 테스트 - 3.3 모듈 패턴](http://www.yes24.com/Product/Goods/33211518)