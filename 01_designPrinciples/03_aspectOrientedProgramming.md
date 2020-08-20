# 03_관점 지향 프로그래밍(Aspect Oriented Programming)

## 정의

![횡단관심사](https://t1.daumcdn.net/cfile/tistory/272F23375892CEF80F)

- 객체지향의 모듈 돌려막기는 한계가 있을 수 있다 => 재사용 가능한 모듈에도 중복된 코드가 있을 수 있다
- 횡단 관심사 : 모듈들을 아우르는 공통된 관심사, 모듈들은 횡단 관심사를 구현하고 해결해야 하기 때문에 모듈들이 완전 독립적이지 못하게 만들 수도 있다.
- 관점 지향 프로그래밍의 목적은 횡단관심사를 모듈화하는 방법을 제시하는 것.
- 메인 비즈니스 로직은 아닌데, 모듈들 간에 돌려쓸 수 있는 일정한 것들을 분리하는 프로그래밍 패러다임
- (단일 책임 범위 내에 있지 않은) 하나 이상의 객체에 유용한 코드를 한데 묶어 **눈에 띄지 않게** 객체에 배포하는 기법

## 구현

뭔가 기능을 하는 모듈을 만들고 거기에 캐싱 기능을 넣고 싶은 상황

```js
// AOP 없는 캐싱
const TravelService = (function(rawWebSer)){

  // 건들일수 없는 클로저 변수들
  const conferenceAirport = 'BOS';
  const maxArrival = new Date()
  const minDeparture = new Date()

  // 캐시 역할을 하는 배열, 역시 접근할수 없다
  let cache = [];

  // 공개 메소드
  return {
    getSuggestedTicket: function(homeAirport) {
      let ticket;
      // 티켓이 한번 발행되었다면, 캐시에 저장되있을테니 캐시에 든거 반환한다
      if (cache[homeAirport]) {
        return cache[homeAirport]
      }
      ticket = rawWebService.getCheapestRoundTrip(
        homeAirport, conferenceAirport, maxArrival, minDeparture
      );
      // 캐시에 저장된다
      cache[homeAirport] = ticket;
      return ticket
    }
  }
})()

TravelService.getSuggestedTicket(attendee.homeAirport)
```

뭐 잘 작동은 하지만 함수의 핵심 로직이라고 할 수 있는 티켓 발행 로직과 캐시하는 로직이 섞여있어 불편하다.  
AOP의 핵심은 **함수 실행을 가로채어 다른 함수를 실행하기 직전이나 직후, 전후에 실행시키는 것이다.**

```js
// AOP 있는 캐싱 => 대충 요런 느낌,,
const TravelService = (function(rawWebSer)){

  // 건들일수 없는 클로저 변수들
  const conferenceAirport = 'BOS';
  const maxArrival = new Date()
  const minDeparture = new Date()

  // 캐시 역할을 하는 배열, 역시 접근할수 없다
  let cache = [];

  _beforeMethod : function(homeAirport) {
   if (cache[homeAirport]) {
        return cache[homeAirport]
      }
    }

  _afterMethod: function(homeAirport) {
      cache[homeAirport] = ticket;
  }

  // 공개 메소드
  return {
    getSuggestedTicket: function(homeAirport) {
      _beforeMethod()
      let ticket;
      // 티켓이 한번 발행되었다면, 캐시에 저장되있을테니 캐시에 든거 반환한다
      ticket = rawWebService.getCheapestRoundTrip(
        homeAirport, conferenceAirport, maxArrival, minDeparture
      );
      // 캐시에 저장된다
      _afterMethod()
      return ticket
    }
  }
})()

```

사실 굳이 전후 시점을 명시적으로 가로챌 필요 없이, 이렇게 메서드를 오버라이딩 하는 방식으로도 구현해볼 수 있을 것이다,,

```js
// 오리지널 메소드를 기억하고 있다가
let originalFunction = Collection.prototype.getNameByISBN;

// 매소드 자체를 수정하는데
Collection.prototype.getNameByISBN = function () {
    // 오리지널 메소드를 실행한후
    let result = originalFunction.apply(this, arguments);
    // 원하는 로직을 수행시킴
    Logger.info(`Retrieving ${result.isbn} - ${result.name} has been succeed`);
    return result;
};
```

## 장점

- 함수를 단순히 유지하며 함수를 더 단일책임에 가깝게 유지한다
- 같은 기능을 하는 코드가 모듈 사이사이에 여기저기 출몰하면 마중에 다른 개발자가 잘못 건드릴 여지도 많고, 동기화가 깨질 가능성이 커진다. 
- AOP는 애플리케이션 설정을 한곳에 집중시킨다. 애스팩트 설정이 단일책임인 함수가 하나만 있으면 부속 기능 전체를 찾을 때 이 함수만 뒤지면 된다.
- jest의 beforeEach나 beforeAll같은 훅이라던지, 프론트엔드 프레임워크의 생명주기에서도 이런 관점을 찾아볼 수 있다.

## reference

- [자바스크립트 패턴과 테스트 - 2.3 애스팩트 툴킷](http://www.yes24.com/Product/Goods/33211518)
- [TOAST 블로그 - 자바스크립트 AOP 맛보기](https://meetup.toast.com/posts/109)
- [관점지향 프로그래밍](https://3months.tistory.com/74)