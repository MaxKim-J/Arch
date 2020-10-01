# 🗂 코드리뷰 레퍼런스 모음

코드리뷰 레퍼런스 될 만한것들(주로 JS) 모아놓은 레포지토리입니다.  
링크를 그대로 가져오거나, 이해가 필요한 것들은 reference 달아서 따로 정리해 놓습니다.

## 규칙

### 업데이트 규칙

1. [TIL]() 레포지토리보다 정리 내용이 깔끔해야 합니다.
2. 코드리뷰 할때 이 레포지토리의 md 문서들을 **실제로 레퍼런스로 쓴다**는 생각을 하면서 정리합니다.
3. md를 작성하고 리드미에 링크를 달아줘서 색인을 만들어 줍니다
4. 각 md의 reference를 소상히 밝힙니다. 레퍼런스의 공신력은 보장되어야 합니다.

## 커밋 규칙

- posting:<제목> [정리|수정|추가]
- refernce:<제목> [정리|수정|추가]
- README: [한 일 업데이트]

## 설계 원칙

- [01_SOLID + DRY원칙](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/01_designPrinciples/01_solidAndDry.md)
- [02_의존성 주입](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/01_designPrinciples/02_dependencyInjection.md)
- [03_관점 지향 프로그래밍](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/01_designPrinciples/03_aspectOrientedProgramming.md)

## 디자인 패턴

- [01_모듈 패턴](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/02_desingPatterns/01_modulePattern.md)
- [02_함수형 상속](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/02_desingPatterns/02_functionalInheritance.md)
- [03_싱글톤 패턴](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/02_desingPatterns/03_singletonPattern.md)
- [04_콜백 패턴](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/02_desingPatterns/04_callbackPattern.md)
- [05_프로미스 패턴](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/02_desingPatterns/05_promisePattern.md)
- [06_관찰자 패턴](https://github.com/MaxKim-J/JS-Code-Review-Reference/blob/master/02_desingPatterns/06_observerPattern.md)
- [07_팩토리 패턴(예정)]()
- [08_전략 패턴(예정)]()
- [09_장식자(데코레이터) 패턴(예정)]()
- [10_프록시 패턴(예정)]()


## 리뷰 레퍼런스

- [왜 css에서 !important 사용을 피해야 하는지](https://uxengineer.com/css-specificity-avoid-important-css/)
- [사파리에서 svg가 깨지는 이유와 대처법](https://jkpark.me/safari/html/css/svg/frontend/2019/06/07/SVG-%EC%82%AC%ED%8C%8C%EB%A6%AC%EC%97%90%EC%84%9C-%ED%9D%90%EB%A6%AC%EA%B2%8C-%EB%B3%B4%EC%9D%B4%EB%8B%A4.html)
- [Vue나 React에서 DOM에 직접 접근을 피하는게 좋은 이유](https://www.danvega.dev/blog/2019/04/18/tips-for-vue-developers-avoid-directly-manipulating-the-dom/) : 데이터 바인딩 등 더 좋은 방법이 존재할 가능성이 있음, 관심사의 분리
