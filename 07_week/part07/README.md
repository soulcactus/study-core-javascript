# 클로저 개념

## 🤘 목표

-   [ ] 클로저의 의미와 원리를 이해한다.

####

## 🕵 클로저의 의미 및 원리 이해

여러 서적에서 클로저를 한 문장으로 요약해 설명하는 부분들을 소개하면 다음과 같습니다.

-   자신을 내포하는 함수의 컨텍스트에 접근할 수 있는 함수
-   함수가 특정 스코프에 접근할 수 있도록 의도적으로 그 스코프에서 정의하는 것
-   함수를 선언할 때 만들어지는 유효범위가 사라진 후에도 호출할 수 있는 함수
-   이미 생명 주기상 끝난 외부 함수의 변수를 참조하는 함수
-   자유변수가 있는 함수와 자유변수를 알 수 있는 환경의 집합
-   로컬 변수를 참조하고 있는 함수 내의 함수
-   자신이 생성될 때의 스코프에서 알 수 있었던 변수들 중 언젠가 자신이 실행될 때 사용할 변수들만을 기억하여 유지시키는 함수

그밖에도 매우 많은 책에서 대부분 위와 같은 정의와 함께 자세한 설명을 이어나가고 있지만, 문장만 놓고 이해할 수 있는 사례보다 그렇지 않은 사례가 많습니다.
이 때문에 본질을 알고 나면 의외로 쉬운 개념인데도 어딘가 갈증이 해소되지 않는 기분을 느끼기 쉬운 개념이 바로 클로저입니다.

MDN은 클로저에 대해 **클로저는 함수와 그 함수가 선언될 당시의 lexical environment의 상호관계에 따른 현상**이라고 소개합니다.
*선언될 당시의 lexical scope*는 지난 파트에서 소개한 실행 컨텍스트의 구성 요소중 하나인 outerEnvironmentReference에 해당합니다.

lexicalEnvironment의 environmentRecord와 outerEnvironmentReference에 의해 변수의 유효범위인 스코프가 결정되고 스코프 체인이 가능해진다고 했습니다.
어떤 컨텍스트 A에서 선언한 내부함수 B의 실행 컨텍스트가 활성화된 시점에는 B의 outerEnvironmentReference가 참조하는 대상인 A의 lexicalEnvironment에도 접근 가능합니다.
A에서는 B에서 선언한 변수에 접근할 수 없지만 B에서는 A에서 선언한 변수에 접근 가능합니다.

여기서 MDN이 설명하는 *상호관계*의 의미를 파악할 수 있습니다.
내부함수 B가 A의 lexicalEnvironment를 언제나 사용하는 것은 아닙니다.
내부함수에서 외부 변수를 참조하는 경우에 한해서만 상호관계, 즉 *선언될 당시의 lexicalEnvironment와의 상호관계*가 중요합니다.

지금까지 파악한 내용에 따르면 클로저란 **어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상**이라고 볼 수 있습니다.
실제 예제 코드를 통해 살펴보겠습니다.

```javascript
const outer = () => {
    let a = 1;

    const inner = () => {
        console.log(++a); // 2
    };

    inner();
};

outer();
```

outer 함수에서 변수 a를 선언했고, outer 함수의 내부함수인 inner 함수에서 a의 값을 1만큼 증가시킨 다음 출력합니다.
inner 함수 내부에서는 a를 선언하지 않았기 때문에 environmentRecord에서 값을 찾지 못하므로 outerEnvironmentReference에 지정된 상위 컨텍스트인
outer의 lexicalEnvironment에 접근해서 다시 a를 찾고, 결과적으로 2가 출력됩니다.
outer 함수의 실행 컨텍스트가 종료되면 lexicalEnvironment에 저장된 식별자들인 a와 inner에 대한 참조를 지웁니다.
그러면 각 주소에 저장돼 있던 값들은 자신을 참조하는 변수가 하나도 없게 되므로 가비지 컬렉터의 수집 대상이 됩니다.
내용을 바꿔 다시 살펴보겠습니다.

```javascript
const outer = () => {
    let a = 1;

    const inner = () => {
        return ++a;
    };

    return inner();
};

const outer2 = outer();

console.log(outer2); // 2
```

이번에도 inner 함수 내부에서 외부 변수인 a를 사용했습니다.
inner 함수를 실행한 결과를 리턴하고 있으므로 결과적으로 outer 함수의 실행 컨텍스트가 종료된 시점에는 a 변수를 참조하는 대상이 없어집니다.
앞의 예제와 마찬가지로 a, inner 변수의 값들은 언젠가 가비지 컬렉터의 수거 대상이 됩니다.

앞의 예제와 위의 예제는 outer 함수의 실행 컨텍스트가 종료되기 이전에 inner 함수의 실행 컨텍스트가 종료돼 있으며,
이후 별도로 inner 함수를 호출할 수 없다는 공통점이 있습니다.
그렇다면 outer의 실행 컨텍스트가 종료된 후에도 inner 함수를 호출할 수 있게 만들면 어떻게 되는지 살펴보도록 하겠습니다.

```javascript
const outer = () => {
    let a = 1;

    const inner = () => {
        return ++a;
    };

    return inner;
};

const outer2 = outer();
console.log(outer2()); // 2
console.log(outer2()); // 3
```

이번에는 함수의 실행 결과가 아닌 inner 함수 자체를 반환했습니다.
그러면 outer 함수의 실행 컨텍스트가 종료될 때 outer2 변수는 outer의 실행 결과인 inner 함수를 참조하게 됩니다.
이후 outer2를 호출하면 앞서 반환된 inner가 실행됩니다.
inner 함수 실행 컨텍스트의 environmentRecord에는 수집할 정보가 없습니다.
outerEnvironmentReference에는 inner 함수가 선언된 위치의 lexicalEnvironment가 참조 복사됩니다.
inner 함수는 outer 함수 내부에서 선언됐으므로, outer 함수의 lexicalEnvironment가 inner 함수의 outerEnvironmentReference에 참조 복사됩니다.
이제 스코프 체이닝에 따라 outer에서 선언한 변수 a에 접근해서 1만큼 증가시킨 후 그 값인 2를 반환하고, inner 함수의 실행 컨텍스트가 종료됩니다.
다시 outer2를 호출하면 같은 방식으로 a의 값을 2에서 3으로 1 증가시킨 후 3을 반환합니다.

inner 함수의 실행 시점에는 outer 함수는 이미 실행이 종료된 상태인데 어떻게 outer 함수의 lexicalEnvironment에 접근할 수 있는 걸까요?
이는 가비지 컬렉터의 동작 방식이기 때문입니다.
가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함하지 않습니다.

위의 예제의 outer 함수는 실행 종료 시점에 inner 함수를 반환합니다.
외부 함수인 outer의 실행이 종료되더라도 내부 함수인 inner 함수는 언젠가 outer2를 실행함으로써 호출될 가능성이 열린 것입니다.
언젠가 inner 함수의 실행 컨텍스트가 활성화되면 outerEnvironmentReference가 outer 함수의 lexicalEnvironment를 필요로 하기 때문에 수집 대상에서 제외됩니다.
그 때문에 inner 함수가 a 변수에 접근할 수 있는 것입니다.

클로저는 어떤 함수에서 선언한 변수를 참조하는 내부함수에서만 발생하는 현상이라고 했습니다.
처음 두 예제에서 일반적인 함수의 경우와 마찬가지로 outer의 경우 변수 a가 가비지 컬렉팅 대상에 포함된 반면, 뒤의 두 예제에서는 제외됐습니다.
어떤 함수의 실행 컨텍스트가 종료된 후에도 lexicalEnvironment가 가비지 컬렉터의 수집 대상에서 제외되는 경우는 마지막 예제와 같이
지역변수를 참조하는 내부함수가 외부로 전달된 경우가 유일합니다.

즉, **어떤 함수에서 선언한 변수를 참조하는 내부함수에만 발생하는 현상**이란 **외부함수의 lexicalEnvironment가 가비지 컬렉팅되지 않는 현상**을 의미합니다.
이를 바탕으로 앞에서 정의한 클로저를 다시 고쳐보면 이렇습니다.
**클로저란 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 B를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상**을 말합니다.
한 가지 주목할 점은 *외부로 전달*하는 것이 곧 return만을 의미하는 것은 아니라는 사실입니다.
예제 코드를 통해 살펴보겠습니다.

```javascript
(() => {
    let interValid = null;
    let a = 0;

    const inner = () => {
        if (++a >= 10) {
            clearInterval(interValid);
        }

        console.log(a);
    };

    interValid = setInterval(inner, 1000);
})();
```

위의 코드는 별도의 외부객체인 window의 메서드(setInterval 또는 setTimeout)에 전달할 콜백함수 내부에서 지역변수를 참조합니다.

```javascript
(() => {
    let count = 0;
    const button = document.createElement('button');

    button.innerText = 'click';

    button.addEventListener('click', function() {
        console.log(++count, 'times clicked');
    });

    document.body.appendChild(button);
})();
```

위의 코드는 별도의 외부객체인 DOM의 메서드(addEventListener)에 등록할 handler 함수 내부에서 지역변수를 참조합니다.
두 상황 모두 지역변수를 참조하는 내부함수를 외부에 전달했기 때문에 클로저에 해당합니다.

####

## 🖥 클로저와 메모리 관리

메모리 누수의 위험을 이유로 클로저 사용을 조심해야 한다거나 심지어 지양해야 한다고 주장하는 목소리도 있지만 메모리 소모는 클로저의 본질적인 특성일 뿐입니다.
*매모리 누수*라는 표현은 개발자의 의도와는 달리 어떤 값의 참조 카운트가 0이 되지 않아, GC의 수거 대상이 되지 않는 경우에는 맞는 표현이지만,
개발자가 의도적으로 참조 카운트를 0이 되지 않게 설계한 경우는 *누수*라고 할 수 없습니다.
과거에는 의도치 않게 누수가 발생하는 여러가지 상황들이 있었지만(순환 참조, IE의 이벤트 핸들러 등)
그중 대부분은 최근의 자바스크립트 엔진에서는 발생하지 않거나 거의 발견하기 힘들어졌으므로 이제는 의도대로 설계한 **메모리 소모**에 대한 관리법만 잘 파악하는 것만으로 충분합니다.

클로저는 어떤 필요에 의해 의도적으로 함수의 지역변수 메모리를 소모하도록함으로써 발생합니다.
그렇다면 그 필요성이 사라진 시점에는 더는 메모리를 소모하지 않게 하면 됩니다.
참조 카운트를 0으로 만들면 언젠가 GC가 수거해 갈 것이고, 이때 소모됐던 메모리도 회수됩니다.
참조 카운트를 0으로 만들기 위해서는 식별자에 참조형이 아닌 기본형 데이터(null 혹은 undefined)를 할당하면 됩니다.
예제 코드를 통해 살펴보도록 하겠습니다.

```javascript
const outer = (() => {
    let a = 1;

    const inner = () => {
        return ++a;
    };

    return inner;
})();

console.log(outer()); // 2
console.log(outer()); // 3

outer = null;
```

```javascript
(() => {
    let interValid = null;
    let a = 0;

    let inner = () => {
        if (++a >= 10) {
            clearInterval(interValid);
            inner = null;
        }

        console.log(a);
    };

    interValid = setInterval(inner, 1000);
})();
```

```javascript
(() => {
    let count = 0;
    const button = document.createElement('button');

    button.innerText = 'click';

    let clickHandler = function() {
        console.log(++count, 'times clicked');

        if (count >= 10) {
            button.removeEventListener('click', clickHandler);
            clickHandler = null;
        }
    };

    button.addEventListener('click', clickHandler);
    document.body.appendChild(button);
})();
```

## 💬 마치며

클로저는 여러 함수형 언어에서 등장하는 보편적인 특성입니다.
자바스크립트 고유의 개념이 아니라서 ECMAScript 명세에도 클로저의 정의를 다루지 않고 있고, 그 때문이라고 할 수 없지만 다양한 문헌에서 제각각 클로저를 다르게 정의 및 설명하고 있는 실정입니다.
더구나 클로저를 설명하는 문장 자체도 이해하기 어려운 단어가 등장하는 경우가 많습니다.
하지만 클로저는 객체지향과 함수형 모두를 아우르는 매우 중요한 개념이므로 잘 숙지하고 활용하도록 해야 합니다.
