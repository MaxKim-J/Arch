# 1. 모듈 패턴

import나 require을 사용하게끔 될 수 있는 기본 원리

## 설명

- 네임스페이스 : 수많은 함수 객체 변수들로 이루어진 코드가 전역 유효범위를 어지럽히지 않고, 애플리케이션이나 라이브러리를 위한 하나의 **representative한 전역 객체**를 만들고 모든 기능을 이 객체에 추가하는 것 => 모듈의 진입점
- 코드에 네임스페이스를 지정해주면 코드 내의 이름 충돌뿐만 아니라 이 코드와 같은 페이지에 존재하는 또다른 서드파티들과의 이름 충돌도 미연에 방지(의존성 해결)
- 모듈 : 전체 애플리케이션의 일부를 독립된 코드로 분리해서 만들어 놓은 것
- 독립된 모듈은 그 자체로 독자적, 그 안에서 모든 의존성을 해결해야 함 => 내부 변수 및 내부 함수를 모두 갖고 있다
- 데이터 감춤 + 임의로 함수를 호출하여 생성 or 선언과 동시에 실행
- 모듈은 복잡한 어플리케이션을 구성하기 위한 블록 역할을 하기도 하지만, 명시적으로 exports 표시되지 않은 모든 내부적인 함수와 변수들을 비공개로 유지하여 정보를 숨기는 중요한 메커니즘(클로저를 통한 private화 + export로 원하는 것들만 내보내기)

## 자바스크립트의 네임스페이스

- 이름이 존재하는 공간, 다른 공간에 존재했기 때문에 식별이 가능한 경우
- 복잡한 프로그램을 개발하거나 협업을 하다보면 전역범위에 이름이 같은 변수, 함수, 객체등을 정의하는 경우가 발생한다. 네임스페이스는 이러한 경우에 발생할 수 있는 충돌을 방지하기 위하여 **이름이 존재하는 공간을 정의하는 기능을 제공한다.**
- 대표적인 예시가 자바의 package. package가 다르면 같은 메소드나 객체명을 가지고 있더라도 다른 개체로 간주된다.
- 자바스크립트에서는 c++나 자바처럼 네임스페이스 기능을 위한 별도의 키워드나 개념을 제공하지 않는다.
- 그래서 의존성 문제나, 상태오염등의 문제가 발생할 수 있음. 그런데 모듈 패턴 등으로 따라할 수는 있다.

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

## common.js 따라하기

### 모듈 로더

노출 모듈 패턴과 마찬가지로 기본적으로 함수로 싸여짐

```js
// 모듈의 내용을 로드하고 이를 private 범위로 감싸 평가하는 함수
// 읽어온 모듈은 module.exports를 변형시킨다
function loadModule(filename, module, require) {
  // 코드 읽고 private화 
  const wrappedSrc = `function(module, exports, require){
    ${
      fs.readFileSync(filename, 'utf8')
      // const someModule = require('./someModule'); => 재귀적으로 다른 모듈을 리졸브
      // module.exports = anotherModule; => module.exports를 사용하여 인자로 준 module 객체 업데이트
    }
  })(module, module.exports, require);`;
  eval(wrappedSrc);
}
```

### require

100프로는 아니고 대충 비슷하게

```js
const require = (moduleName) => {
  console.log(`Required invoked for module: ${moduleName}`);

  // 1. 경로 알아내기 
  const id = require.resolve(moduleName)

  // 2. 모듈이 이미 로드된 경우 캐시를 사용함
  if(require.cache[id]) {
    return require.cache[id].exports
  }

  // 3. 모듈의 메타데이터 : 모듈의 최초 로드를 위한 환경설정
  const module = {
    exports:{},
    id:id
  };

  // 4. module 객체의 캐시 업데이트
  require.cache[id] = module;

  // 5. 모듈 소스코드는 해당 파일에서 읽어오며 코드는 앞에서 살펴본 방식대로 평가됨 
  // 모듈은 module.exports 객체를 조작하거나(파일 안에서) 대체하여 public API를 내보낸다
  // 인자로는 파일네임의 아이디, 모듈 객체, 그리고 require을 내부에서 사용할 수 있도록 인자로 제공
  loadModule(id, module, require);

  return module.exports;
}

require.cache = {};
require.resolve = (moduleName) => {
  // moduleName에서 moduleId를 확인하기
  // 경로를 알아냄
}
```

- module.exports 변수에 할당되지 않는 한 모듈 내부의 모든 항목은 private이다.
- 변수 exports는 module.exports의 초기 값에 대한 참조일 뿐임. 본질적으로 이 값은 모듈이 로드되기 전에 만들어진 간단한 객체 리터럴임
- export가 참조하는 객체에만 새로운 속성을 추가할 수 있음(exports.뭐 = 뭐)
- export 변수의 재할당은 module.export의 내용을 변경하지 않기 때문에 아무런 효과가 없다. 그것은 변수 자체만을 재할당하는 거,,
- require 함수가 동기적으로 작동한다는 것도 알아야함. 간단하게 자체 로직만으로 모듈의 내용을 반환하니 콜백이 필요하지는 않음
- 모듈을 비동기적으로 초기화해야하는 과정이 필요한 경우에는 모듈이 미래 시점에 비동기적으로 초기화되기때문에 미처 초기화되지 않은 모듈을 정의하고 익스포트해버릴 수도 있음. 그러니깐 그거는 require을 사용한다고 해서 사용할 준비가 된다는 보장이 없음

### resolve

require나 import에 문자열만 넣은 것 같은데 어떻게 모듈이 잘 불러와지는 것임???

```js
// 종종 이런 경우를 봤을 것임
const someModule = require('someModule');
import react from 'react';
```

- node.js 모듈은 로드되는 위치에 따라 다른 버전의 모듈을 로드할 수 있도록 함
- resolve함수는 **모듈 이름을 입력으로 사용하여 모듈 전체의 경로를 반환함**
- resolve 알고리즘 개요
  - 파일 모듈 : moduleName이 `/`로 시작되면 이미 모듈에 대한 절대경로라고 간주되어 그대로 반환됨. `./`로 시작하면 상대경로로 간주되고, 이는 요청한 모듈로부터 시작되어 계산
  - 코어 모듈 : moduleName이 경로명이 아니면 코어 node 모듈 내에서 검색함(fs같은거)
  - 패키지 모듈 : 일치하는 코어 모듈이 없는 경우, 요청 모듈의 경로에서 시작하여 디렉토리 구조를 탐색하여 올라가면서 node_modules 디렉토리를 찾고 안에서 일치하는 모듈을 찾는 일을 계쏙함. 알고리즘은 신기하게도 파일 시스템의 루트에 도달할때까지 디렉토리 트리를 올라가면서 다음 node_modules 디렉토리를 탐색하면서 다음 node_modules 디렉토리를 탐색하여 계속 일치하는 모듈을 찾음
- 요 알고리즘은 node.js의 의존성 관리의 견고함을 뒷받침하는 핵심적인 부분. 충돌 혹은 버전 호환성 문제없이 어플리케이션에서 수백 수천개의 패키지를 가질 수 있게 됨
- require에 구현된 캐싱은 일정판 패키지 내에서 동일한 모듈이 필요할 때는 어느정도 동일한 인스턴스가 항상 반환되는 것을 보장함.

## reference

- [module pattern](https://webclub.tistory.com/5)
- [자바스크립트 패턴과 테스트 - 3.3 모듈 패턴](http://www.yes24.com/Product/Goods/33211518)