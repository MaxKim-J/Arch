# 02_함수형 상속 & 멍키 패칭

상속이라는 패러다임을 느슨하고 자유롭게 구현할 수 있는 자바스크립트의 매력...  

## 함수형 상속

상속의 생성자 반복 문제를 해결하는 방법  
**상속자로부터 데이터를 감춘다**  

```js
// 전역 네임 스페이스
const AnimalKingdom = AnimalKingdom || {};

// 모체
AnimalKingdom.marsupial = function(name, isNocturnal) {
  const instanceName = name, instanceIsNoctutnal = isNocturnal;
  return {
    getName:function() {
      return instanceName;
    },
    getIsNocturnal: function() {
      return instanceIsNocturnal
    }
  }
}

// 상속된 객체
AnimalKingdom.kangaroo = function(name) {
  // 클로저로 모체를 가져오고 오버라이딩을 한다
  const baseMarsupial = AnimalKingdom.marsupial(name, false);
  baseMarsupdial.hop = function() {
    return baseMarsupial.getName() + '가 껑충 뛴다';
  };
  return baseMarsupial
}

// 상속된 객체를 바탕으로 새로운 인스턴스
const mike = AnimalKingdom.kangaroo('마이크');

// 모체 + 오버라이딩된 메소드 모두에 접근이 가능
mike.getName();
mike.getIsNocturnal();
mike.hop();
```

- 전역 네임스페이스 안에 함수로서 객체를 선언하고 필요한 디펜던시도 함수 안에 선언하고(클로저) 메소드를 밖으로 빼낸다
- 상속을 할 인스턴스를 해당 전역 네임스페이스 안에 프로퍼티로 선언하고
- 전역 네임스페이스에서 상속의 모체가 되는 함수를 불러와서 새 프로퍼티 스코프 안에 클로저로 메소드를 가져온다
- 가져오고 거기다가 프로퍼티를 붙이고는(오버라이딩) 변형된 메소드 뭉치를 반환한다 
- 생성 로직을 super할 필요도 없다. 직접 사용하기 때무네
- 이 과정에서 기존의 생성 로직은 변형이 안된다 개방/폐쇄 원칙이 충실히 반영된 결과임..

## 멍키 패칭

- 추가 프로퍼티를 객체에 붙임. 기존 객체의 프로퍼티를 다른 객체에 붙여옴  
- 일종의 메서드 빌림

## reference

- [자바스크립트 패턴과 테스트 - 3.3.7 함수형 상속](http://www.yes24.com/Product/Goods/33211518)
