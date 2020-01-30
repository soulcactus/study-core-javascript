# 클로저

## 🧐 클로저의 의미 및 원리 이해

MDN에서는 클로저는 "함수와 그 함수가 선언될 당시의 lexical environment의 상호관계에 따른 현상" 이라고 소개하고있습니다. 

'선언될 당시의 LexicalEnvironment'는 실행 컨텍스트의 구성 요소 중 하나인 outerEnvironmentReference에 해당합니다. 

LexicalEnvironment의 environmentRecord와 outerEnvironmentReference에 의해 변수의 유효범위인 스코프가 결정되고 스코프 체인이 가능해집니다.

* 환경레코드(environmentRecord) - 모든 지역 변수를 프로퍼티로 저장하고 있는 객체. this가 참조하는 값 등의 정보도 저장됨.

* 외부환경참조(outerEnvironmentReference) - 외부 코드와 연관되어 있음.

어떤 컨텍스트 A에서 선언한 내부함수 B의 실행 컨텍스트가 활성화된 시점에는 B의 outerEnvironmentReference가 참조하는 대상인 A의 LexicalEnvironment에도 접근이 가능합니다. A에서는 B에서 선언한 변수에 접근할 수 없지만 B에서는 A에서 선언한 변수에 접근 가능합니다.

여기까지 파악한 내용에 따르면 클로저란,  "**어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상**" 이라고 볼 수 있겠는데요. 아직 와닿지 않으니 예제를 통해 좀 더 명확히 알아보겠습니다. 예제를 보기 전 위의 내용을 더 잘 이해하기 위해 밑에서 스코프와 스코프체인, outerEnvironmentReference에 대해 잠깐 짚고 넘어가겠습니다.

### **스코프, 스코프 체인, outerEnvironmentReference**

스코프란 **식별자에 대한 유효범위**입니다. 어떤 경계 A의 외부에서 선언한 변수는 A의 외부뿐 아니라 A의 내부에서도 접근이 가능하지만, A의 내부에서 선언한 변수는 오직 A의 내부에서만 접근할 수 있습니다. **'식별자의 유효범위'를 안에서부터 바깥으로 차례로 검색해나가는 것**을 스코프 체인이라고 합니다. 그리고 이를 가능하게 하는 것이 바로 LexicalEnvironment의 두 번째 수집자료인 outerEnvironmentReference입니다.

**1) 스코프 체인**

outerEnvironmentReference는 **현재 호출된 함수가 선언될 당시의 LexicalEnvironment를 참조**합니다. 과거 시점인 '선언될 당시'에 주목해야합니다. 

예를 들어, A함수 내부에 B함수를 선언하고 다시 B함수 내부에 C 함수를 선언한 경우, 함수 C의 outerEnvironmentReference는 함수 B의 LexicalEnvironment를 참조합니다. 그리고 함수 B의 LexicalEnvironment에 있는 outerEnvironmentReference는 다시 함수 B가 선언 되던 때(A)의 lexicalEnvironment를 참조하게됩니다. 

'선언시점의 LexicalEnvironment'를 계속 찾아 올라가면 마지막엔 전역 컨텍스트의 LexicalEnvironment가 있을 것입니다. 

또한 각 outerEnvironmentReference는 오직 자신이 선언된 시점의 LexicalEnvironment만 참조하고 있으므로 가장 가까운 요소부터 차례대로만 접근할 수 있고 다른 순서로 접근하는 것은 불가능할 것입니다.

이런 구조적 특성 덕분에 여러 스코프에서 동일한 식별자를 선언한 경우에는 **무조건 스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근 가능**하게 됩니다.

아래 예시를 통해 좀 더 구체적으로 알아보겠습니다

```javascript
// (시작)
var a = 1; // (1)

var outer = function() { // (3)
  var inner = function() { // (4) / (6)
    console.log(a); // (7)
    var a = 3; // (8)
  }; // (9)

  inner(); // (5)
  console.log(a); // (10)
} // (11)

outer(); // (2)
console.log(a) // (12)

// -- 결과 --
// undefined
// 1
// 1
```

1. (시작) : 전역 컨텍스트가 활성화 됩니다. 전역 컨텍스트의 environmentRecord에 { a, outer } 식별자를 저장하고, 전역 컨텍스트는 선언 시점이 없으므로 전역 컨텍스트의 outerEnvironmentReference에는 아무것도 담기지 않습니다.

2. (1) : 전역 스코프에 있는 변수 a에 1을, outer에 함수를 할당합니다.

3. (2) : outer 함수를 호출합니다. 이에 따라 전역 컨텍스트의 코드는 이 줄에서 임시 중단되고, outer 실행 컨텍스트가 활성화되어 2번째 줄로 이동합니다.

4. (3) : outer 실행 컨텍스트의 environmentRecord에 { inner } 식별자를 저장합니다. outerEnvironmentReference에는 outer 함수가 선언될 당시의 LexicalEnvironment가 담기며, outer 함수는 전역 공간에서 선언됐으므로 전역 컨텍스트의 LexicalEnvironment를 참조복사합니다. [ Global, { a, outer } ] 첫 번째는 실행 컨텍스트의 이름, 두 번째는 environmentRecord 객체입니다.

5. (4) : outer 스코프에 있는 변수 inner에 함수를 할당합니다.

6. (5) : inner 함수를 호출합니다. 이에 따라 outer 실행 컨텍스트의 코드는 이 줄에서 임시 중단되고, inner 실행 컨텍스트가 활성화 되어 세 번째 줄로 이동합니다.

7. (6) : inner 실행 컨텍스트의 environmentRecord에 { a } 식별자를 저합니다. outerEnvironmentReference에는 inner 함수가 선언될 당시의 LexicalEnvironment가 담기며 inner 함수는 outer 함수 내부에서 선언됐으므로 outer 함수의 LexicalEnvironment, 즉 [ outer, { inner } ]를 참조복사합니다.

8. (7) : 식별자 a에 접근하고자 합니다. 현재 활성화 상태인 inner 컨텍스트의 environmentRecord에서 a를 검색하여 발견하지만 여기에는 아직 할돵된 값이 없습니다. (undefined 출력)

9. (8) : inner 함수 실행이 종료됩니다. inner 실행 컨텍스트가 콜 스택에서 제거되고, 바로 아래의 outer 실행 컨텍스트가 다시 활성화되면서 5)에서 중단했던 줄의 다음으로 이동합니다.

10. (9) :  inner 함수 실행이 종료됩니다. inner 실행 컨텍스트가 콜 스택에서 제거되고, 바로 아래의 outer 실행 컨텍스트가 다시 활성화되면서 (5)에서 중단했던 줄의 다음으로 이동합니다.

11. (10) : 식별자 a에 접근하고자 합니다. 활성화된 실행 컨텍스트의 LexicalEnvironment에 접근하며 첫 요소의 environmentRecord에서 a가 있는지 찾아보고, 없으면 outerEnvironmentReference에 있는 environmentRecord로 넘어가는 식으로 계속해서 검색합니다. 예제에서는 두 번째, 즉 전역 LexicalEnvironment에 a가 있으니 그 a에 저장된 값 1을 반환. (1 출력)

12. (11) : outer 함수 실행이 종료됩니다. outer 실행 컨텍스트가 콜 스택에서 제거되고, 바로 아래의 전역 컨텍스트가 다시 활성화 되면서 2)에서 중단했던 줄의 다음으로 이동합니다.

13. (12) : 식별자 a에 접근하고자 합니다. 현재 활성화 상태인 전역 컨텍스트의 environmentRecord에서 a를 검색하며 바로 a를 찾을 수 있습니다(1 출력). 이로써 모든 코드의 실행이 완료되어 전역 컨텍스트가 콜 스택에서 제거되고 종료됩니다.


outerEnvironmentReference는 **해당 함수가 선언된 위치의 LexicalEnvironment를 참조**합니다. 

코드 상에서 어떤 변수에 접근하려고 하면 현재 컨텍스트의 LexicalEnvironment를 탐색해서 발견되면 그 값을 반환하고, 발견하지 못할 경우 다시 outerEnvironmentReference에 담긴 LexicalEnvironment를 탐색하는 과정을 거칩니다.

전역 컨텍스트의 LexicalEnvironment까지 탐색해도 해당 변수를 찾지 못하면 undefined를 반환합니다.

### **클로저**

다시 클로저로 돌아와, 외부함수에서 변수를 선언하고 내부함수에서 해당 변수를 참조하는 형태의 코드를 보겠습니다.

```javascript
// 외부 함수의 변수를 참조하는 내부 함수(1)

var outer = function() {
  var a = 1;
  var inner = function() {
    console.log(++a);
  };
  inner();
}
outer();

// -- 결과 --
// 2
```

위의 예제에서는 outer 함수에서 변수 a를 선언했고, outer의 내부함수인 inner 함수에서 a의 값을 1만큼 증가시킨 다음 출력합니다. 

inner 함수 내부에서는 a를 선언하지 않았기 때문에 environmentRecord에서 값을 찾지 못하므로 outerEnvironmentReference에 지정된 상위 컨텍스트 outer의 LexicalEnvironment에 접근하여 다시 a를 찾습니다. 네 번째 줄에서는 2가 출력됩니다.

outer 함수의 실행 컨텍스트가 종료되면 LexicalEnvironment에 저장된 식별자들(a, inner)에 대한 참조를 지웁니다. 그러면 각 주소에 저장돼 있던 값들은 자신을 참조하는 변수가 하나도 없게 되므로 가비지 컬렉터의 수집 대상이 될 것입니다.

내용을 조금 바꾼 밑 예제를 이어서 보겠습니다.

```javascript
// 외부 함수의 변수를 참조하는 내부 함수(2)

var outer = function() {
  var a = 1;
  var inner = function() {
    return ++a;
  }
  return inner(); // inner 함수 결과 반환
};

var outer2 = outer();
console.log(outer2);

// -- 결과 --
// 2
```

위 코드에서도 inner 함수 내부에서 외부 변수인 a를 사용했습니다. 그런데 6번째 줄에서는 inner 함수의 실행 결과를 리턴하고 있으므로 결과적으로 outer 함수의 실행 컨텍스트가 종료된 시점에는 a변수를 참조하는 대상이 없어집니다. a, inner 변수의 값들은 언젠가 가비지 컬렉터에 의해 소멸하게됩니다. 

위의 (1), (2) 예제는 outer 함수의 실행 컨텍스트가 종료되기 이전에 inner 함수의 실행 컨텍스트가 종료돼 있으며, 이후 별도로 inner 함수를 호출할 수 없다는 공통점이 있습니다.

그렇다면 outer의 실행 컨텍스트가 종료된 후에도 inner 함수를 호출할 수 있게 만들면 어떻게 될까요? 밑의 (3) 예제 코드를 보겠습니다.

```javascript
// 외부 함수의 변수를 참조하는 내부 함수(3)

var outer = function() {
  var a = 1;
  var inner = function() {
    return ++a;
  }
  return inner; // inner 함수 자체 반환
};

var outer2 = outer();
console.log(outer2());
console.log(outer2());

// -- 결과 --
// 2
// 3
```

이번에는 6번째 줄에서 inner 함수의 실행 결과가 아닌 inner 함수 자체를 반환했습니다.
그러면 outer 함수의 실행 컨텍스트가 종료될 때 outer2 변수는 outer의 실행 결과인 inner 함수를 참조하게 될 것입니다. 이후 9번째 줄에서 outer2를 호출하면 앞서 반환된 함수인 inner가 실행됩니다.

inner 함수 실행 컨텍스트의 environmentRecord에는 수집할 정보가 없습니다. outerEnvironmentReference에는 inner 함수가 선언된 위치의 LexicalEnvironment가 참조복사됩니다. inner 함수는 outer 함수 내부에서 선언됐으므로, outer 함수의 LexicalEnvironment가 담기게 됩니다. 

이제 스코프 체이닝에 따라 outer에서 선언한 변수 a에 접근하여 1만큼 증가시킨 후 그 값인 2를 반환하고, inner 함수의 실행 컨텍스트가 종료됩니다. 다시 outer2를 호출하면 같은 방식으로 a의 값을 2에서 3으로 1 증가시킨 후 3을 반환합니다.

❓❓❓

여기서, inner 함수의 실행 시점에는 outer 함수는 이미 실행이 종료된 상태인데 어떻게 outer 함수의 LexicalEnvironment에 접근할 수 있는 걸까요? 

이는 **가비지 컬렉터의 동작 방식** 때문입니다. 가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함시키지 않습니다. 

위의 예제에서 outer 함수는 실행 종료 시점에 inner 함수를 반환합니다. 외부함수인 outer의 실행이 종료되더라도 내부 함수인 inner함수는 언젠가 outer2를 실행함으로써 호출될 가능성이 열린 것입니다. 언젠가 inner 함수의 실행 컨텍스트가 활성화되면 outerEnvironmentReference가 outer 함수의 LexicalEnvironment를 필요로 할 것이므로 수집 대상에서 제외됩니다. 이 때문에 inner 함수가 이 변수에 접근할 수 있는 것입니다. 

👀👀

클로저는 어떤 함수에서 선언한 변수를 참조하는 내부 함수에서만 발생하는 현상이라고 했습니다. 예제(3)의 경우는 변수 a가 가비지 컬렉터 수집 대상에서 제외됐습니다. 

이처럼 함수의 실행 컨텍스트가 종료된 후에도 LexicalEnvironment가 가비지 컬렉터의 수집 대상에서 제외되는 경우는 예제(3)과 같이 지역변수를 참조하는 내부 함수가 외부로 전달된 경우가 유일합니다. 

"**어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상**"이란 "**외부 함수의 LexicalEnvironment가 가비지 컬렉팅되지 않는 현상**"을 말하는 것이겠죠.

이를 바탕으로 클로저의 정의를 다시 정리해보면 이렇습니다.

클로저란 **어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상**을 말합니다

다른 서적들에서는 클로저를 이렇게 정의하고 있습니다.

* 함수를 선언할 때 만들어지는 유효범위가 사라진 후에도 호출할 수 있는 함수 - <<자바스크립트 닌자 비급. 인사이트>>

* 이미 생명 주기가 끝난 외부 함수의 변수를 참조하는 함수 - <<인사이드 자바스크립트.한빛미디어>>

* 자신이 생성될 때의 스코프에서 알 수 있었던 변수들 중 언젠가 자신이 실행될 때 사용할 변수들만을 기억하여 유지시키는 함수 - <<함수형 자바스크립트 프로그래밍.인사이트>>

❗❗❗

주의할 점이 한 가지 있습니다. 바로 '외부로 전달'이 곧 return만을 의미하는 것은 아니라는 점입니다. 밑의 코드를 통해 확인해보겠습니다.

```javascript
// return 없이도 클로저가 발생하는 다양한 경우
// setInterval / setTimeout

(function() {
  var a = 0;
  var intervalId = null;
  var inner = function() {
    if(++a >= 10) {
      clearInterval(intervalId);
    }
    console.log(a);
  };
  intervalId = setInterval(inner, 1000);
})

// -- 결과 --
// 1
// 2
// 3
// 4
// 5
// 6
// 7
// 8
// 9
// 10
```

위의 예제는 별도의 외부객체인 window의 메서드(setTimeout 또는 setInterval)에 전달할 콜백 함수 내부에서 지역변수를 참조합니다. 

```javascript
// return 없이도 클로저가 발생하는 다양한 경우
// eventListener

(function() {
  var count = 0;
  var button = document.createElement("button");
  button.innerText = "Click";
  button.addEventListener("click", function() {
    console.log(++count, "times clicked!");
  });
  document.body.appendChild(button);
})

// -- 결과 --
// 1 "times clicked!"
// 2 "times clicked!"
// 3 "times clicked!"
// 4 "times clicked!"
// ...
```
위의 두 번째 예제에서는 별도의 외부객체인 DOM의 메서드(addEventListener)에 등록할 handler 함수 내부에서 지역번수를 참조합니다. 

두 예시 코드 모두 지역변수를 참조하는 내부함수를 외부에 전달했기 때문에 클로저입니다.

## 🧐 클로저와 메모리 관리

메모리 누수의 위험을 이유로 클로저 사용을 조심해야 한다거나 심지어 지양해야 한다고 주장하는 사람들도 있지만 메모리 소모는 클로저의 본질적인 특성일 뿐, 오히려 이러한 특성을 정확히 이해하고 잘 활용할 수 있도록 노력해야 한다고 책에서 저자가 말하고 있습니다. 

'메모리 누수'라는 표현은 개발자의 의도와 달리 어떤 값의 참조 카운트가 0이 되지 않아 GC(가비지 컬렉터)의 수거 대상이 되지 않는 경우에는 맞는 표현이지만 개발자가 의도적으로 참조 카운트를 0이 되지 않게 설계한 경우는 '누수'라고 할 수 없을 것입니다. 의도대로 설게한 '메모리 소모'에 대한 관리법만 잘 파악해서 적용하는 것으로도 충분합니다.

관리 방법은 간단합니다.

클로저는 어떤 필요에 의해 의도적으로 함수의 지역변수가 메모리를 소모하도록 함으로써 발생합니다. 그 필요성이 사라진 시점에서 더는 메모리를 소모하지 않게 해주면 됩니다. 참조 카운트를 0으로 만들면 언젠가 GC가 수거해갈 것이고, 소모됐던 메모리가 회수될 것입니다. 

참조 카운트를 0으로 만드는 방법은 식별자에 참조형이 아닌 기본형 데이터(보통 null이나 undefined)를 할당하면 됩니다. 

밑의 예제 코드는 위에서 살펴본 return 없이도 클로저가 발생하는 경우의 예시들에 메모리 해제 코드를 추가한 코드입니다.

```javascript
// return에 의한 클로저의 메모리 해제

var outer = (function() {
  var a = 1;
  var inner = function() {
    return ++a;
  };
  return inner;
})();

console.log(outer());
console.log(outer());
outer = null; // outer 식별자의 inner 함수 참조를 끊음
```

```javascript
// setInterval에 의한 클로저의 메모리 해제

(function() {
  var a = 0;
  var intervalId = null;
  var inner = function() {
    if(++a >= 10) {
      clearInterval(intervalId);
      inner = null; // inner 식별자의 함수 참조를 끊음
    }
    console.log(a);
  };
  intervalId = setInterval(inner, 1000);
})
```

```javascript
// eventListener에 의한 클로저의 메모리 해제

(function() {
  var count = 0;
  var button = document.createElement("button");
  button.innerText = "Click";
  
  var clickHandler = function() {
    console.log(++count, "times clicked!");
    if(count >= 10) {
      button.removeEventListener("click", clickHandler);
      clickHandler = null; // clickHandler 식별자의 함수 참조를 끊음.
    }
  };

  button.addEventListener("click", clickHandler);
  document.body.appendChild(button);
}
```





