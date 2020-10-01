# 07_팩토리 패턴

객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 인스턴스를 만들지는 **서브 클래스에서 결정하게 만든다.** 즉 팩토리 메소드 패턴을 사용하면 클래스의 인스턴스를 만드는 일을 서브클래스에서 맡기는 것(JS로 따지면 함수에서 임의로 인스턴스를 생성하는 것)

## Node에서의 적용

- 프로토타입에서 직접 새 객체를 만드는 대신 팩토리를 호출하면 여러 면에서 훨씬 편리하고 유연해진다.
- 팩토리는 객체 생성을 구현과 분리할 수 있도록 만든다. 새로운 인스턴스의 생성을 감사서 우리가 하는 방식에 더 많은 유연성과 제어력을 제공
- 팩토리의 소비자는 인스턴스 생성이 수행되는 방법에 대해서는 전적으로 알 필요가 없다. new 연산자를 사용하면 객체 하나를 생성하는데 한 가지 특정한 방법으로만 코드를 바인드할 수 있지만 더 제약이 없는 방식으로 객체를 만들 수 있다.

```js
// 간단한 팩토리
// 객체(인스턴스)를 만드는 로직을 캡슐화
function createImage(name) {
  return new Image(name);
}
const image = createImage('photo.jpeg');
// const image = new Image(name);
```

- new를 사용하면 하나의 특정한 유형의 객체만을 코드에 바인딩할 수 있다. 팩토리는 대신 더 많은 유연성을 제공한다.
- 인자에 따라서 다른 인스턴스를 만들게끔 해줄수도 있구...
- 팩토리는 또한 생성된 객체의 생성자를 노출시키지 않고 객체를 확장하거나 수정하지 못하도록 한다(익명으로 냅다 내보내기 때문에). 각 생성자를 비공개로 유지한채 팩토리만 내보내는 방법 활용 가능하다.

## 캡슐화를 강제하는 매커니즘

- 클로저랑 궁합이 좋다. 자바스크립트에서는 private 변수를 선언할 수 없기 때문에 캡슐화를 적용하는 유일한 방법은 함수 범위(스코프)와 클로저를 사용하는 것인데 팩토리는 private 변수처럼 쓸 수 있다.
- private 변수 역할을 하는 클로저 앞에 _나 $를 앞에 붙이는 방법같은게 있나봄

```js
// person factory
function createPerson(name) {
  // 클로저 변수 => private 변수처럼 사용
  const privateProperties = {};

  //! 멤버변수와 메소드를 따로 분리하고 있다는게 포인트일듯
  // 공개 객체에 메소드 정의
  // 클로저 변수를 따로 만들고, 이를 참조하는 메소드를 품은 객체를 따로 만드는 이유는
  // person의 name이 비어있을 수 없도록 강제하기 위해서
  const person = {
    setName:name => {
      if(!name) throw new Error('A person mush have a name');
      privateProperties.name = name;
    },
    getName: () => {
      return privateProperties.name;
    }
  }

  // 무족권 name은 존재하게 됨
  person.setName(name);
  return person;
}
```

- 또한 자스의 동적 타이핑 때문에 리턴값이 조건에 따라 천차만별이어도 괜찮음 => 덕타이핑
- **구현으로부터 객체의 생성을 분리한다** => 팩토리 소비자는 함수의 안이 어떻게 되어있는지 알 필요가 없다. 인터페이스에 적합한 객체를 받기만 하면 된다.

## 합성이 가능한 팩토리 함수(stampit.js)

- 향상된 팩토리 함수를 만들기 위해 조합될 수 있는 특정 유형의 팩토리 함수
- 복잡한 클래스 계층 구조를 만들지 않고도 다양한 소스에서 동작하면서 속성을 상속하는 객체를 만들때 유용
- 되게 희한한 라이브러리라서 알면 좋을거같아 기록해봅니다.

```js
import stampit from 'stampit'

const Character = stampit({
  props: {
    name: null,
    health: 100
  },
  init({ name = this.name }) {
    this.name = name
  }
})

// 합체
const Fighter = Character.compose({ // inheriting
  props: {
    stamina: 100
  },
  init({ stamina = this.stamina }) {
    this.stamina = stamina;    
  },
  methods: {
    fight() {
      console.log(`${this.name} takes a mighty swing!`)
      this.stamina--
    }
  }
})

// 합체
const Mage = Character.compose({ // inheriting
  props: {
    mana: 100
  },
  init({ mana = this.mana }) {
    this.mana = mana;    
  },
  methods: {
    cast() {
      console.log(`${this.name} casts a fireball!`)
      this.mana--
    }
  }
})

const Paladin = stampit(Mage, Fighter) // as simple as that!

const fighter = Fighter({ name: 'Thumper' })
fighter.fight()
const mage = Mage({ name: 'Zapper' })
mage.cast()
const paladin = Paladin({ name: 'Roland', stamina: 50, mana: 50 })
paladin.fight()
paladin.cast()

console.log(Paladin.compose.properties) // { name: null, health: 100, stamina: 100, mana: 100 }
console.log(Paladin.compose.methods) // { fight: [Function: fight], cast: [Function: cast] }
```