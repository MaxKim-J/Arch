# 05_프로미스 패턴

## 설명

- Promise는 비동기 작업과 그 결과를 갖고 해야할 일을 캡슐화한 객체로서, 작업 완료 시 이 객체에 캡슐화한 콜백을 호출한다.
- Promise 생성자의 인자는 비동기 작업을 감싼 함수, 이 함수는 두 인자 resolve와 reject를 받는다. Promise가 resolve, 혹은 reject처리되면 둘 중 한 함수가 호출된다
- Promise를 pending 상태에서 벗어나게 하려면 then을 사용한다. 첫번째 콜백은 이행값을 받고, 다른 함수는 거부 이유를 받는다.
- 프라미스는 값을 가지고 동기적으로 프라미스를 resolve한다고 할지라도, onFulfilled와 onRejected 함수에 대한 비동기적 호출을 보장한다. => 어쨋든 두 함수는 resolve후에 실행된다.

## 테스팅 관점

- 프로미스가 귀결된 시점에 값을 평가해야한다는 것을 염두해두어야 한다.

## reference

- [자바스크립트 패턴과 테스트 - 6. 프라미스 패턴](http://www.yes24.com/Product/Goods/33211518)