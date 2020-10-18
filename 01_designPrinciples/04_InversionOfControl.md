# 04_제어의 역전(Inversion Of Control)

고차원 모듈은 저차원 모듈에 의존하면 안 된다. 이 모듈 모두 다른 추상화된 것에 의존해야 한다. 
추상화 된 것은 구체적인 것에 의존하면 안 된다. 구체적인 것이 추상화된 것에 의존해야 한다. - 로버트 마틴

## 의존성(Dependency)

![의존 이미지](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbSHQ5T%2Fbtqz8YGPMTD%2FLep8mGYmFR678izmDODldK%2Fimg.png)

- 의존성 : 구성 요소들이 서로 의존하는 성질
- Client A가 Service B를 의존하고 있을 때, Service B가 변경되면 컴파일이 안되거나 예상치 못한 동작을 하는 등의 영향을 받게 된다.
- 이러한 의존성은 A를 재사용하기 어렵게 만들기 때문에 A는 컴포넌트가 될 수 없다.
- Component : 소스 코드의 아무런 수정 없이 다른 프로젝트에서 바로 재사용이 가능한 수준의 모듈
- 의존성이 계속 얽히면 Leaf에 존재하는 Class 말고는 재사용할 수 있는 코드가 전혀 없다는 뜻을 의미함.
- 컴포넌트가 될 수 있는 클래스가 몇개인지, UI나 Data Source등이 변경되었을 때 각 class간의 영향도 어느 정도인지는 개발에 있어서 중요
- 고차원 모듈이 저차원 모듈을 의존하고 저차원 모듈이 다시 고차원 모듈을 의존하는 것을 의존성 부패라고 함.

## 제어의 역전(Dependency Inversion Principle)

![의존성 제거](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FTMAIz%2FbtqAbmJdmLq%2F8UkEkmPwJlI9wvYdnSLRvK%2Fimg.png)

- 의존성 부패를 제거하기 위한 일반적인 디자인 방법
- **클래스 안에서 다른 생성자를 사용해 직접 인스턴스를 만들어 멤버변수로 쓰면** 영원히 의존에서 자유롭지 못하다. 멤버변수로 불러오는 client는 재사용할 수 없는 모듈이 되고, 일종의 오염이 일어난다.
- 옳은 방법은 일단 멤버변수에 interface로(즉 추상화된 것으로) 타입 지정을 해주는 것이다. 이때 멤버변수는 인스턴스 그 자체가 아니라 interface에 의존하게 되는데, 이것이 **구체적인 것(인스턴스)이 아닌 추상적인 것(인터페이스)에 의존하라는** 원칙을 따르게 된다.
- interface에 의존하고 있는 멤버변수는, 해당 interface를 extend하고 있는 어떤 모듈이든 멤버변수로 사용이 가능하다. client 클래스는 한 클래스에서의 의존성에서 벗어나서, 다른 인자에도 열러있는 클래스가 된다.
- 하지만 DIP를 적용했다고 해도 완벽하게 컴포넌트가 되지는 못하는데, 여전히 작지만 의존성이 남아있기 때문이다. 팩토리나 클래스에 대해서.

```ts
// 의존성 오염
class SwitchButton {
  constructor() {
    this.lamp = new Lamp()
    this.isOn = false
  }
}

// interface에 의존
class SwitchButton {
  constructor() {
    this.lamp:SwitchMachine = new Lamp()
    this.isOn = false
  }
}

// Factory를 구현하면 특정 interface를 준수하는 클래스를 받는 상황을 보장한다.
class SwitchButton {
  constructor() {
    this.lamp:SwitchMachine = Factory.getObject()
    this.isOn = false
  }
}
```

## 의존성 주입(Dependency Injection)

```ts
class SwitchButton {
  constructor(machine:SwitchMachine) {
    this.lamp = machine
    this.isOn = false
  }
}
```

- 외부로부터 의존성을 주입받는 것
- 멤버 변수로 concrete한 dependency를 없애서 외부로부터 변경사항에 대한 영향도가 매우 적어졌다. 컴포넌트의 모습을 갖추게 된 것이다.

## 의존 관계 역전의 원칙

- SOLID 원칙 중에 하나
- **변화하기 쉬운 것 보단 변화하기 어려운 것에 의존하라는 원칙**
- 변화하기 쉬운 것과 그렇지 않은 것 사이에서의 판단이 필요하다는 말
- 이걸 잘 지키면 다른 SOLID 원칙인 **개방-폐쇄 원칙**도 준수할 수 있다.
- 두 개의 약간 다른 클래스가 있다면 공통점을 찾아 하나의 상위 클래스나 인터페이스로 묶고, 그것을 다른 클래스의 의존성으로 사용하는 것은, 상위 클래스가 하위 클래스에 의존하고 있는 현상이다(?).

```ts
// 여기서 개발자가 노트북으로 코딩을 한다는 사실은 잘 변하지 않는다.
// 우리는 거기에 의존해야한다.
interface LapTop {
  turnOn(): void;
}

class MacBook implements LapTop {
  public turnOn() {}
}

class Gram implements Laptop {
  public turnOn() {}
}

class Programmer {
  // 그래서 클래스가 인자에서 참조하는 인터페이스로 LapTop을 참조할 수 있다
  // 하위 클래스가 상위 클래스에 의존해야 하는데, 여기서는 상위 클래스의 구현이 하위 클래스에 달려있는 꼴이 된다.
  constructor(private laptop:Laptop) {}
  public programming() {
    this.laptop.turnOn();
  }
}

const programmer1:Programmer = new Programmer(new MacBook());
programmer1.programming()

const programmer2:Programmer = new Programmer(new Gram());
programmer2.programming()
```

## 제어의 역전(Inversion Of Control)

![제어으 역전](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbCCWkv%2FbtqAjNkWkqX%2Fzkp2Hl3ey1duXVyV4MC7v1%2Fimg.png)

- 어떠한 일을 하도록 만들어진 프레임워크(컨테이너)에 제어의 권한을 넘김으로써 클라이언트 코드가 신경써야 할것을 줄이는 전략.
- **프레임워크의 메소드가 사용자의 코드를 호출한다.** 큰 컨테이너에서 의존성이 알아서 resolve된다.
- 오브젝트는 자신이 사용할 오브젝트를 스스로 생성하거나 선택하지 않는다.
- **오브젝트는 자신이 어떻게 생성되고 사용되는지 알 수 없다**
- service가 client를 최대한 모르게 하는게 확장성에 좋다!
- IoC가 적용되지 않은 일반적인 프로그램의 흐름은 entry point에서 다음에 사용할 오브젝트를 결정하고, 생성하고, 생성된 오브젝트의 메서드를 호출하고, 그 오브젝트 메서드 안에서 또 다음에 사용할것을 결정하고 호출하고......(안됨)
- 즉 service를 사용하는 client쪽에서 모든 것을 제어하고 있는 구조이다. 제어의 역전은 이러한 제어의 흐름을 inversion하는 것이다.

## reference

- spring의 대표개념이다 보니... 자바를 배우면 더 잘 이해할 수 있지 않을까
- [develogs - 제어의 역전(Inversion of Control, IoC) 이란?](https://develogs.tistory.com/19)
- [Alvin - TypeScript와 typedi로 의존성 주입 이해하기](https://medium.com/@HoseungJang/typescript%EC%99%80-typedi%EB%A1%9C-%EC%9D%98%EC%A1%B4%EC%84%B1-%EC%A3%BC%EC%9E%85-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-5d83ef1977f9)