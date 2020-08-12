# 싱글톤 패턴

## 설명

- 인스턴스가 오직 1개만 생성되야 하는 경우에 사용되는 패턴
- 싱글톤 패턴은 이 객체가 **특정 네임스페이스에서 유니크함**을 보장해야한다
- 중요한 설정 파일 같은 경우(넘 자바 이야기긴 한데) 객체가 여러개가 생성되면 설정 값이 변경될 위험이 생길 수 있음

## 예시

### 자바의 예시

```java
// 뭐 스레드 세이프하지 않다고 한다
public class Singleton {
    // 이른 초기회, 클래스 로더가 초기화하는 시점에서 인스턴스를 메모리에 등록
    // 인스턴스를 private static으로 등록해버림
    private static Singleton uniqueInstance = new Singleton();

    // 생성자를 private화 하여 부를 수 없게 만든다
    private Singleton() {}

    // 오직 스태틱 변수로만 참조할 수 있다.
    public static Singleton getInstance() {
      return uniqueInstance;
    }
}
```

- 자바의 싱글톤은 인스턴스 자체 하나 만들어서 private static으로 들고다니게만 하고 + 생성자를 무력화시키는 방법으로 철저하게 만들어진다. 
- 자바에서의 스레드 세이프함은 중요하다(멀티 스레딩 방식에서도 잘 동작해야 한다, 뭐 동시성 이런이야기) => 컴파일 시점에 인스턴스를 생성하는 것이 아니라 인스턴스가 필요한 시점에 요청하여 동적 바인딩(런타임시에 성격이 결정됨)을 통해 인스턴스를 생성하는 방식을 사용

### JS의 예시

```js
var singleton = (function() {

  // 클로저 변수 : 얘한테는 직접 접근할수가 엄슴, 바꿀수도 없구
  var instance;
  var aName = 'hello';

  // 클로저 함수 : 비공개 메서드 물론 얘도 접근못함
  function initiate() {
    return {
      a: aName,
      b: function() {
        alert(a);
      }
    };
  }

  // 공개 객체
  return {
    getInstance: function() {
      // 이미 한번 호출했다면 새 객체를 반환하지 않는다 
      // 일종의 캐싱이랄까
      if (!instance) {
        instance = initiate();
      }
      return instance;
    }
  }
})();   // 즉시 호출

var first = singleton.getInstance();
var second = singleton.getInstance();
console.log(first === second); // 같다
```

```js
// es6이상의 클래스로 구현

class Singleton{
  // 클래스의 정적 변수 : 인스턴스 사이에서는 공유댐
  static instance;
 
  constructor(){
    if(instance) return instance;
    this.name = "heecheolman";
    this.age = 24;
    instance = this;
  }
}
 
var firstSingleton = new Singleton();
var secondSingleton = new Singleton();
 
console.log(firstSingleton === secondSingleton); // true
console.log(instance === firstSingleton); // true
```

- 객체 리터럴은 자바스크립트 싱글톤 패턴의 가장 단순한 구현체다 
- 객체 리터럴을 선언하면 저 객체는 단 하나밖에 존재하지 않으므로 싱글톤 패턴을 따른 것이다 물론 이 때는 데이터 감춤같은 기능은 없다
- 데이터를 비공개로 만들어야 싱글턴이라고 할 수 있는데 이때는 뭐 결국 또 클로저를 사용한다.
- 다른 객체 생성패턴과 달리 다른 함수를 생성하기 위해 호출할 함수도 없고, new 키워드로 함수를 찍어낼 일도 없다(그냥 가져다 쓰면 됨)
- 스레드 세이프는 딱히 자바스크립트에서 고려사항이 되지 못한다 => 기본적으로 싱글 스레드 언어라서..
- instance에는 접근할 수 없는게 맞는데(인스턴스를 다른걸로 대채한다거나 하는건 불가능) 인스턴스의 프로퍼티는 공개된 상태라서 바꿀 수 있긴 하다

## reference

- [싱글턴 패턴](https://medium.com/webeveloper/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36)
- [자바스크립트 싱글톤 패턴](https://velog.io/@recordboy/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8B%B1%EA%B8%80%ED%86%A4-%ED%8C%A8%ED%84%B4)
- [디자인 패턴(싱글턴, 모듈, 생성자)](https://www.zerocho.com/category/JavaScript/post/57541bef7dfff917002c4e86)

