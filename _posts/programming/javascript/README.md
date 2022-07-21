## 초급 단계
- 루프, if 구문, try/catch등 같은 기본 프로그래밍 문법을 아는것.
- 함수가 정의되고 할당되는 다양한 방법과 더욱이 익명함수를 포함하는 함수 정의법에 대한 이해.
- 기본 스코프 원리, global(window) 스코프 vs 객체 스코프.(closure는 제외)
- context의 역할과 'this' 변수 사용의 이해.
- 객체에 대한 인스턴스화와 선언한는 것 뿐만아니라 함수를 인스턴스화 하고 선언하는 다른 방법들에 대한 이해.
- '<', '>', '==', '==='와 같은 자바스크립트 비교연산자들, falsy가 무엇인지, 객체와 문자열을 비교하는 작업 뿐만아니라 변환 작업들의 이해.
- 개체의 어트리뷰트와 함수들의 인덱싱이 실제 배열과 어떻게 다른지 이해(object 리터널 vs array 리터널).
 
## 중급단계
- timer들, 그것들이 어떻게 동작하고, 언제 어떻게 timer들이 유용한지, 더욱이 비동기 메서드 실행에 대한 이해.
- 컨텍스트 조작과 function argument passing 을 위한 'call', 'apply'같은 function application과 콜백 개념에 대한 깊은 이해.
- JSON표기법과 'eval' 함수에 대한 이해
- 클로저에 대한 이해, 코드상에서 어떤 효과를 내는지, 또한 클로저가 private 변수들을 생성하기 위해 어떻게 사용되는지, (function(){})() 호출과 함께.
- AJAX와 객체 직렬화.
 
## 고급단계
- 메소드의 'arguments' 변수와 이것이 어떻게 arguments를 통해 함수 오버로드에 사용되는지에 대한 이해. length 속성과 arguments.callee를 통해 재귀적 호출하기. 비록 jQuery(v1.4 이후)와 Dojo에서 이것을 이용하고 있지만, ECMAScript 5 Strict Mode에서 지원하지 않고 있으므로 arguments.callee 사용은 위험할 수 있다고 지적한다.
- self-memoizing functions, currying, 그리고 partially applied functions들과 같은 클로저에 대한 고급이해.
- Function과 html DOM에 대한 prototype 확장, prototype 체인, 코딩을 최소화하기 위해 자바스크립트의 기본 객체와 함수(예. Array)를 사용하는 법.
- 개체 타입과 instanceof 와 typeof의 사용.
- 정규 표현식과 표현식 편집하기
- With구문과 왜 With 구문을 사용하면 안되는지 이해.
- 무엇보다 가장 어려운 부분은 깨끗하고, 강력하고, 빠르고, 유지보수 가능하고 그리고 크로스 브라우징 되는 이 모든 수단들을 한데 모으는 법을 아는것이다.
 
## 추가적으로 알아야 하는 사항
- Dom과 그것을 효율적으로 조종하는 방식, 즉 노드의 추가, 삭제, 변경과 더욱이 텍스트 노드의 핸들링. 이것은 document fragments 같은 툴들을 통해 브라우저의 re-flow를 최소화 하는것도 포함한다.
- 크로스 브라우저 방식으로 DOM element로 부터 정보를 추출하는 것(예를들어 style, position, 등등.). 이와같은 것들은 jQuery 또는 Dojo 같은 프레임워크에서 훌륭하게 구현하고 있지만, CSS와 style 태그, computing style에서 정보 추출의 차이점을 이해는 것은 매우 중요합니다.
- 크로스 브라우저 이벤트 처리, 바인딩, 언바인딩 그리고 어떻게 원하는 callback context 이뤄내는지. 이런 것들은 프래임워크를 통해 훌륭하게 처리 되지만 IE와 W3C 표준 브라우저의 차이점을 이해해야 합니다.
- Expandos 대 attribute setting, 둘사이의 성능 차이와 존재하는 네이밍의 불일치.
- DOM 노드 추출을 위한 정규표현식.
- 효과적인 브라우저 기능 디텍팅과 장애 완화. 

## [자바스크립트 개발자라면 알아야 할 33가지 개념](https://velog.io/@jakeseo_me/series/33conceptsofjavascript)
