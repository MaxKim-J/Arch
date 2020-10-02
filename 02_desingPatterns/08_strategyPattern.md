# 08_전략 패턴

객체들이 할 수 있는 행위 각각에 대한 전략 클래스를 생성하고, 유사한 행위들을 캡슐화하는 인터페이스를 정의하여 객체의 행위를 동적으로 바꾸고 싶은 경우 직접 행위를 수정하지 않고 전략을 바꿔주기만 함으로써 행위를 유연하게 확장하는 디자인 패턴

## node에서의 적용

- 컨텍스트라고 불리는 객체를 사용하여 변수 부분을 상호 교환 가능한 개별 전략이라는 객체들로 **추출**, 연산 로직의 변형을 지원함
- 컨텍스트는 일련의 알고리즘의 공통 로직을 구현하는 반면, 개별 전략은 입력값, 시스템 구성 혹은 사용자 기본 설정 같은 다양한 요소들을 컨텍스트의 동작에 적용할 수 있도록 변경 가능한 부분을 구현.
- 컨텍스트 객체가 다양한 전략들을 교체 가능한 부품처럼 교체하고 연결시킬 수 있어야 한다.
- 복잡한 조건문으로 계속 나열해야 하는 경우를 피할 수 있다
- 무언가가 바뀌었을 때마다 메소드 자체를 계속 수정해야 하는 건, SOLID의 개방-폐쇠 원칙에 위반된다.
- 의존성 역전(주입) 원칙을 반영한 디자인 패턴
- passport의 로그인 전략!

## 예제

파일 형식마다 다른 strategy를 견지하는 config 만들기

```js
const fs = require('fs');
const objectPath = require('object-path');

class Config {
  // 데이터를 분석하고 직렬화하는 알고리즘을 나타내는 변수 strategy를 입력으로 받음
  constructor(strategy) {
    this.data = {};
    this.strategy = strategy;
  }

  get(path) {
    return objectPath.get(this.data, path);
  }

  set(path, value) {
    return objectPath.set(this.data, path, value);
  }

  // strategy의 프로퍼티를 참조해서 무언가를 함
  read(file) {
    console.log(`${file} 디시리얼라이징 하겠음`);
    this.data = this.strategy.deserialize(fs.readFileSync(file, 'utf-8'));
  }

  save(file) {
    console.log(`${file} 시리얼라이징 하겠음`);
    fs.writeFileSync(file, this.strategy.serialize(this.data));
  }
}
module.exports = Config;


// 갈아끼울 부품들

const jsonStrategy = {
  deserialize: data => JSON.parse(data),
  serialize: data => JSON.stringify(data, null, ' ')
}

const iniStrategy = {
  deserialize: data => ini.parse(data),
  serialize: data => ini.stringify(data, null, ' ')
}

// 생성자
// 제공된 파일에 따라 동적으로 strategy를 선택하는 식으루다가 추상화를 한번 더 할수도 있겠다
const jsonConfig = new Config(jsonStrategy);
const iniConfig = new Config(iniStrategy);
```
## reference

- [victolee - 전략 패턴](https://victorydntmd.tistory.com/292)
- [Node.js 디자인패턴 - 전략 패턴](http://www.yes24.com/Product/Goods/65050060)
