# 09_장식자 패턴

**객체에 동적으로 새로운 책임을 추가한다.** 서브클래스를 생성하는 것보다 융통성 있는 방법을 제공한다. 가끔 전체 클래스에 새로운 기능을 추가할 필요는 없지만, 개별적인 객체에 새로운 책임을 추가할 필요가 생김. 

일반적인 방식으로는 상속을 이용하면 되겠지마는, 일단 클래스 새로 만들기 귀찮고 사용자는 구성요소를 어떻게 장식해야할지 동적으로 제어할 수 없다(기존에 만들어진 클래스를 가져다 쓰는 것이기 때문에). 

더 나은 방법은 추가할 구성요소를 감싸는 것. 이렇게 무엇인가 감싸는 객체를 장식자라고한다. 장식자는 자신이 둘러싼 요소, 구성요소가 갖는 인터페이스를 자신도 동일하게 제공하므로 장식자의 존재는 이를 사용하는 사용자에게 감춰짐.

탈부착 가능한 책임을 정의하고 싶을 때, 상속이 비효율적일때, 객체의 타입과 호출 가능한 메소드를 그대로 유지하면서 객체에 새로운 책임을 추가할때 사용한다.

## Node에서의 적용

- 데코레이터는 기존 객체의 동작을 동적으로 증강시키는 구조적 패턴이다. 이 동작을 동일한 클래스의 모든 객체에 추가되지 않고 명시적으로 데코레이트한 인스턴스에만 추가되기 때문에 고전적인 상속과는 다른 양상이다.
- 가로채서 객체를 바꾼다는 점에서 프록시랑 비슷한 면이 있는데 가장 큰 차이점은 이미 존재하는 메소드를 modify하거나 하지 않고 추가만 한다는 점이다.

## 예제

프록시랑 구현 목적은 거의 똑같음

### 컴포지션

```js
function decorate(component) {

  const proto = Object.getPrototypeOf(component);

  function Decorator(component) {
    this.component = component;
  }

  Decorator.prototype = Object.create(proto)

  // 새로운 메소드 추가
  Decorator.prototype.greetings = function() {
    return 'Hi~';
  }

  Decorator.prototype.hello = function() {
    return this.component.hello.apply(this.component, arguments);
  }

  return new Decorator(component)
}
```

### 객체증강

```js
// 직접 새 메소드 연결
// 프록시는 재할당이었고, 데코레이터는 그냥 추가만 한다
function decorate(component) {
  component.greetings = () => {

  };
  return component;
}
```

## 타입스크립트의 데코레이터 문법

자스에는 아직 없고 타입스크립트에 실험적 기능으로 존재하는 데코레이터 문법에 대해서 추가로 정리해보자

- 기본적으로 함수
- 팩토리를 만들어야 한다면 고차함수
- 클래스, 메서드, 프로퍼티, 접근자, 파라미터 선언에 이용된다
- 데코레이터 표현식은 런타임에 함수로서 호출됨

### 프로퍼티

```ts
// 조건을 인자로 받아서 조건에 따라 writable
function readOnly(condition?: () => boolean) {
  return function decorator(target, name): any {  
    return {
      writable: condition ? condition() : true,
    }
  }
}

class Product {
  @readOnly(() => {
    return new Date > new Date(2020, 0, 1)
  })
  name: string;

  @readOnly()
  price: number;

  constructor(name: string, price: number) {
    this.name = name;
    this.price = price;
  }
}

const p1 = new Product('foo', 2000);

p1.name = 'foo';
p1.price = 3000;
```

### 클래스

- extends를 통해 새로운 프로퍼티를 추가하거나, 기존 프로퍼티를 오버라이드 하는 기능 제공
- 인자로는 클래스의 생성자 함수-컨스트럭터가 전달된다. 클래스 데코레이터 함수에서는 새로운 클래스만을 반환할 수 있고, 함수 이외의 값은 무시된다.
- 고차함수 : 추가적인 인자, 컨스트럭터, 클래스(최종 결과물)

```ts
// 오버라이드, 추가 데코레이터
// 여기서의 제네릭 : 객체를 리턴하는 생성자 인터페이스를 확장한 것
function classDecorator<T extends {new(...args:any[]):{}}>(constructor:T) {
  return class extends constructor {
      newProperty = "new property";
      hello = "override";
  }
}

// 클래스에 필요한 의존성을 constructor로 주입하는 데코레이터
// 인스턴스화 되는 시점에서 필요한 의존성을 주입받는다
function inject(...depNames) {
  return function<T extends {new(...args: any[]): {}}> (constructor: T) {
    return class extends constructor {
      // 컨스트럭터를 오버라이딩
      constructor(...args: any[]) {
        const deps = depNames.reduce((deps, name) => ({
          ...deps,
          [name]: dependencyPool[name],
        }), {});
        // 서브 클래스에서 부모 생성자 호출하여 멤버변수 승계
        // super의 인자는 부모클래스 생성자의 인자
        super(deps);
      }
    }
  }
}

@inject('dep1', 'dep2')
class Product {
  constructor(deps) {
    console.log('product dependency is', deps);
  }
}

@classDecorator
class Greeter {
  property = "property";
  hello: string;
  constructor(m: string) {
      this.hello = m;
  }
}

console.log(new Greeter("world")); 
// {property: "property", hello: "override", newProperty: "new property"}
```

### 메서드

```js
class Product {
  setPrice() {
    console.log('setPrice');
  }
}

// 모체가 되는 객체(this), 프로퍼티
const descriptor = Object.getOwnPropertyDescriptor(Product.prototype, 'setPrice');
console.log(descriptor);
// {value: ƒ, writable: true, enumerable: false, configurable: true}
console.log(descriptor.value === Product.prototype.setPrice);
// true
```

- 메서드 데코레이터는 메서드의 property descriptor를 수정하여 메서드를 확장한다
- 객체의 프로퍼티들을 정교하게 정의할 수 있는 ES5 스펙
- 요거는 ES5 스펙이므로 tsconfig.json의 target이 es5보다 낮아버리면 사용이 불가능함
- 첫번째 인자 : static 메서드라면 클래스의 생성자 함수, 인스턴스 메서드라면 클래스의 prototype 객체를 받는다 => 근데 사실 prototype 객체를 받으면 생성자 함수(constructor)도 접근가능한거 아니냐
- 두번째 인자: 메서드 이름 => 이건 어따써먹지?
- 세번째 인자: 메서드의 property Descriptor => 요 객체의 value 프로퍼티를 수정하는 방식으로 동작을 바꿔줄 수 있다.

```ts
function logging(target, name, descriptor) {
  // 그냥 이렇게 기억하는것도 나쁘지 않을거같음 = 따로 빼면 this를 잃어먹는다
  const originMethod = descriptor.value;
  // 해당 메서드를 직접 modify
  descriptor.value = function(...args) {
    // 아..그렇네 이건 originMethod를 이렇게 실행시켜줘야겠네
    // 엄청 프록시스럽다
    // 여길 기점으로 전후 동작을 modify하는 느낌이다
    // 여기서의 this는 descriptor => this.그메소드 이런식으로 해도 되겟다 분리하지말고
    const res = originMethod.apply(this, args);
    console.log(`${name} method arguments: `, args);
    console.log(`${name} method return: `, res);
    return res;
  }
}
```

### 접근자

```ts
function accessorDeco(accessorType) {
  console.log('decorator for', accessorType);
  return function(target, name, descriptor) {
    // 뭐시기뭐시기
  }
}

class Product {
  // 의미적 private
  _price: number = 1000;

  @accessorDeco('getter')
  get price() {
    return this._price;
  }

  // Compile Error
  // Decorators cannot be applied to multiple get/set accessors of the same name.
  @accessorDeco('setter')
  set price(p) {
    this._price = p;
  }
}

const p = new Product();
// decorator for getter
```

- 언더바로 의미적으로 private임을 드러내는 getter setter
- getter, setter에 적용되는 데코레이터임. 메서드 데코레이터와 인자가 동일함
- 하나의 프로퍼티에 대한 get, set 메서드에 동일한 데코레이터가 적용될 수 없음

## reference

- [haeguri - 타입스크립트 데코레이터](https://haeguri.github.io/2019/08/25/typescript-decorator/)
- [John Grib - 데코레이터 패턴:객체에 동적으로 새로운 책임을 추가한다](https://johngrib.github.io/wiki/decorator-pattern/)
- [Node.js 디자인패턴 - 장식자 패턴](http://www.yes24.com/Product/Goods/65050060)
