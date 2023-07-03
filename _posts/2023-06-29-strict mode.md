---
title: "[JS] 엄격모드(Strict Mode) 정리"
author: hangyeoldora
categories: [All, Javascript]
tags: [javascript, js, ES5, ES6]

---

# *Strict Mode

## 엄격모드란?

<br/>

​     일반적으로 React를 사용해본 경험이 있다면 index.js에서 `<React.StrictMode/>`또는 `<StrictMode/>`를 본 적이 있을 것이다. 이는 애플리케이션 내의 문제를 알아내기 위한 것인데, hook에 대한 올바른 사용과 더불어 자손들에대한 부가적인 검사 등 유효성 검사를 담당한다. 

자바스크립트에서도 React와 같이 엄격모드(Strict mode)를 사용할 수 있는데, 이는 ES5(2009)에 등장하였다. 기본적으로는 엄격모드가 아닌 느슨한 모드(Sloppy Mode)이기 때문에 엄격모드 선언이 필요한데, 그것이 바로 `'use strict;'` 다. 
<br/>

## 1. 기능 및 사용법

- 기존에 무시되던 에러를 throwing

- JS 엔진 최적화를 어렵게하는 실수를 잡아주며, 가끔 느슨한 모드보다 빠르게 실행

- 차기 적용되는 ECMA 문법을 금지

- 적용할 스크립트 앞에 `use strict;` 선언

- 전체 스크립트 또는 부분 함수에 적용이 가능
  
  -> 전체 스크립트 : eval., function, event handler, setTimeout 등 전달된 문자열

- 블럭문 {}, 컨텍스트에는 적용이 불가

- ES5의 JS 모듈의 전체 콘텐츠는 자동으로 엄격 모드가 적용

- 엄격모드는 구문과 런타임의 동작을 변경

### 실수를 <u>에러로 throw</u>

1. 변수 생성 시, 선언이 없을 경우
- 기존에는 var 또는 let or const 선언 없이도 `변수명=값` 을 지정하고 변수를 호출하면 값이 정상적으로 나왔다. 하지만, 엄격 모드에서는 변수에 대한 선언 없이는 에러를 호출한다.
  
  ```javascript
  // 기본
  x=5;
  x; // 5 출력
  
  // 엄격 모드
  'use strict';
  x=5;
  x; // 참조에러 발생, not defined
  ```
  
  => 꼭 선언을 해줘야 함
2. 동일한 이름의 파라미터가 있을 경우
   
   - 하나 이상의 파라미터명이 같으면 에러가 발생
     
     ```javascript
     'use strict';
     const testFunc = (a,a,c) =>{
         return a+b+c;
     }
     console.log(testFunc(1,2,3));
     // 에러 발생
     ```
   
   - 기본 모드에서 마지막으로 중복된 인수가 이렇게 지정된 인수를 숨기는데 이러한 인수들은 arguments[i]를 통해 접근이 가능
     
     => 이러한 숨김처리는 개발 이치에 맞지 않음

3. 변수 또는 함수 등 선언 후에 삭제하려고 할 때, 에러 발생
   
   - 선언 후에 delete하려고 하면 에러가 발생함
     
     ```javascript
     // IIFE를 통한 예시
     (()=>{
       'use strict';
       let x = 5;
       delete x; // SyntaxError 발생
     })();
     ```
     
     => function 또한 동일한 에러 발생
     
     > ※ IIFE(Immediately Invoked Function Expression, 즉시 실행 함수 표현)
     > 
     > - 정의되자마자 즉시 실행하는 함수로서 익명함수이다.
     >   IIFE에 대한 자세한 내용은 해당 <a href="">포스트</a> 를 참고바란다.

4. 삭제할 수 없는 프로퍼티를 삭제하려고 하는 경우
   
   - 프로퍼티 삭제 시, 에러 발생
     
     ```javascript
     delete Object.prototype // TypeError
     ```

5. 예외를 발생시키는 에러
   
   - NaN은 쓸 수 없는 전역 변수이며, NaN에 할당하는 일반적인 코드는 아무것도 하지 않는다. 하지만, 엄격 모드에서 NaN에 할당하면 에러가 발생한다.
      또한, 쓸 수 없는 전역 프로퍼티, getter-only 프로퍼티, 확장 불가 객체에 프로퍼티를 할당하는 경우 예외-에러 발생
     
     ```javascript
     // undefined
     let undefined = 5 // TypeError
     
     // getter-only
     let obj = (get x() {return 17;});
     obj.x = 5; // Error
     
     // 확장 불가 객체
     let fixed = {}
     Object.preventExtensions(fixed);
     fixed.new Prop = "ohai"; // Error
     ```

6. 8진수 구문 및 이스케이프 문자 사용
   
   - 8진수를 그대로 사용하는 것은 에러를 발생
     
     ```javascript
     // 8진수
     let oct1 = 010
     
     // 이스케이프 문자 사용
     let oct2 = "/010";
     ```
     
     => ECMA2015에서는 접두사에 "0o"를 붙여서 8진수를 지원

7. 숫자 앞에 0 사용
   
   - 정렬하려고 숫자를 사용하는 경우가 있는데, 이럴 경우 에러가 발생
     
     ```javascript
     let num1 = 012 +
                345 +
                678;
     // Error 발생
     ```

8. 원시값에 프로퍼티 설정 금지
   
   - 기본 모드(느슨한 모드)에서는 되지만, 엄격 모드에서는 TypeError를 발생시킨다. 
     
     ```javascript
     // primitive
     false.true = "";
     (14).sailing = "home"
     ```
   
   

9. 실수로 글로벌 변수 생성
   
   - 일반 JS에서 변수를 잘못 입력하면 전역 객체에 대한 새 속성이 생성되고 동작을 한다. 이렇게 전역 변수를 생성하는 할당은 엄격 모드에서는 에러를 발생하게 한다.
     

### 보안 증가

1. this로 함수에 전달된 값은 객체가 되지 않음
   
   - 일반적인 함수에서 this는 언제나 객체인데, 이러한 자동 박싱은 성능 비용뿐만 아니라 전역 객체가 노출되어 위험하다.
     
     -> 엄격모드에서는 this가 boxed된 객체가 아니며, 정의가 되지 않은 경우 `undefined`로 나온다.
     
     ```javascript
     'use strict';
     function test() { return this; };
     console.assert(test()) // undefined
     console.assert(test.call(2)) // 2
     console.assert(test.apply(undefined)) // undefined
     console.assery(test.bind(true)) // true
     ```
   
   - 특정 this를 명세하기 위해서는 .call, .apply,s .bind 메서드를 통해 가능
     
     -> 더 이상 window 객체를 this로 참조가 불가능

2. 더 이상 스택을 따라 올라가는 것이 불가능

3. 인수는 더 이상 해당 함수의 호출 변수에 대한 접근을 제공하지 않음
