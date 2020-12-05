# 04_콜백 패턴

## 설명

- 콜백 : 나중에 실행할 부차 함수에 인자로 넣는 함수
- 여기서 콜백이 실행될 나중 시점이 부차함수의 실행 완료 이전이면 **동기**, 반대로 실행 완료 이후면 **비동기**라고 한다.
- 프라미스나 이런건 비동기 콜백에만 적용되는 것이다
- 작업 결과를 전달하기 위해 호출되는 함수, 비동기 작업을 처리할때는 반드시 필요

## 방식

- 연속 전달 방식 : 결과를 인자로 전달하는 방식, 콜백이 어떤 작업을 마치고 수행될 때
- 비연속 전달 방식 : 딱히 연산 결과를 전달하지는 않고, 어떤 동작을 정의해서 수행할 뿐(Array.prototype.map의 콜백처럼)

## 예측할 수 없는 함수

- 동기와 비동기 콜백의 짬뽕은 결과를, 또는 작업의 순서를 예측하지 못하는 코드를 만든다
- 노드 리더 아이작 슈월처 : 이런 함수는 마치 [Zalgo를 풀어놓은듯한 함수](https://blog.izs.me/2013/08/designing-apis-for-asynchrony)

```js
const fs = require('fs');
const cache = {};

// 동기 비동기 짬뽕함수
function inconsistentRead(filename, callback) {
  // 캐시 있으면 동기로 동작 아니면 파일 읽어서 비동기로 동작
  if(cache[filename]){
    callback(cache[filename]);
  } else {
    // 비동기 동작하는 파일리더
    fs.readFile(filename, 'utf8', (err,date) => {
      // 캐싱
      cache[filename] = data;
      callback(data);
    })
  }
}

function createFileReader(filename) {
  // 클로저 변수
  const lisnters = [];
  inconsistentRead(filename, value => {
    // listners에 들어간 함수 실행
    listners.forEach(listner => listner(value));
  })
  return {
    // 리스너는 여기서 인자로 들어오는 콜백함수다
    // listners에 함수를 넣음
    onDataReady: listner => listners.push(listner)
  }
}

const reader1 = createFileReader('data.txt');
reader1.onDataReady(data => {
  // 출력, 비동기 로직의 콜백 => 파일리딩이 끝나야
  console.log('first call data');
  // 이경우는 같은 파일을 읽었는데, 캐시가 있을 경우 동기 출력됨 
  const reader2 = createFileReader('data.txt');
  reader2.onDataReady(data => {
    // 안출력 : 동기로 진행되는 로직의 비동기 로직
    // 배열에는 들어가지만 호출되지 않는다
    console.log('second call data')
  })
})
```

- 그러니까 API의 동기 또는 비동기 특성을 명확하게 정의하는 것이 필수적임
- 모듈을 나눌 때 **동기인 거랑 비동기인 거랑** 일관성있게 모으고, 하나만 하는게 이상적이다.

## 규칙

- 콜백은 맨 마지막 위치에 => 코드의 가독성 때문
- Node.js에서 CPS 함수에 의해 생성된 오류는 항상 콜백의 첫번째 인자로 전달, 실제 결과는 두번째 인수에서부터 전달. 에러가 없다면 첫번째 인자는 null이나 undefined

## 테스팅 관점

- **익명 콜백**은 콜백만 따로 떼어낼 방법이 없으니 단위 테스트가 힘들다
- 익명 함수는 디버깅을 어렵게 만든다, 익명 함수는 정의 자체가 이름없는 함수라서 호출 스택에 식별자를 표시할 수 없기 때문
- 콜백 함수에 식별자를 부여하는 방식으로 이를 해결할 수 있다
- 또는 콜백에 스파이를 붙여서, 콜백이 몇번 호출되었는지 제대로 실행되었는지를 검증한다.
- 테스트 스파이 예시 : [jest.spyOn](https://www.daleseo.com/jest-fn-spy-on/)

## reference

- [Node.js 디자인패턴 - 콜백 패턴](http://www.yes24.com/Product/Goods/65050060)
- [자바스크립트 패턴과 테스트 - 콜백 패턴](http://www.yes24.com/Product/Goods/33211518)
