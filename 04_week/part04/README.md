# 스코프와 스코프 체인

## 🤘 목표

-   [ ] 스코프와 스코프 체인을 이해한다.

####

## 🔗 스코프

```javascript
function a() {
    var str1 = 'a';

    console.log(str2); // ReferenceError: str2 is not defined

    function b() {
        var str2 = 'b';

        console.log(str1); // a
        console.log(str2); // b
    }
}

a();
```

스코프란 식별자에 대한 유효범위입니다. 함수 b의 외부에서 선언한 변수 str1은 b의 외부뿐만 아니라 b의 내부에서도 접근 가능하지만, b의 내부에서 선언한 변수 str2는 오직 b의 내부에서만 접근 가능합니다.
이와 같은 스코프 개념은 대부분의 언어에 존재하며, 자바스크립트도 예외는 아닙니다만 ES5까지의 자바스크립트에서는 특이하게도 전역 공간을 제외하면 오직 함수에 의해서만 스코프가 생성됩니다.
ES6에서는 블록에 의해서도 스코프 경계가 발생하므로 다른 언어와 훨씬 비슷해졌습니다.
다만, 블록 스코프는 지난 번 파트에서 살펴봤듯이 var로 선언한 변수에 대해서는 적용하지 않고(자동으로 undefined를 할당하기 때문)
오직 새 키워드인 let과 const, class, strict mode에서의 함수 선언 등에 대해서만 적용됩니다.
때문에 ES6에서는 둘을 구분하기 위해 함수 스코프, 블록 스코프라는 용어를 사용합니다.
이러한 식별자의 유효범위를 안에서부터 바깥으로 차례로 검색해 나가는 것을 *스코프 체인*이라고 합니다.
그리고 이를 가능하게 하는 것이 지난 번 파트에서 살펴봤던 LexicalEnvironment의 두번째 수집 자료인 outerEnvionmentReference입니다.

####

## ⛓️ 스코프 체인

outerEnvionmentReference는 현재 호출된 함수가 **선언될 당시의 LexicalEnvironment를 참조**합니다.
예제를 통해 살펴보겠습니다.

```javascript
var a = 1;

var outer = function() {
    var inner = function() {
        console.log(a); // 1
    };

    inner();
    console.log(a); // 1
};

outer();
console.log(a); // 1
```

먼저 outer 함수가 실행되기 전까지의 전역의 LexicalEnvironment를 살펴보겠습니다.
전역 환경이므로 OutterEnviromentReference가 존재하지 않습니다.

```javascript
LexicalEnvironment = {
    EnviromentRecord: {
        DeclartionEnvironmentRecord = {
            a: 1,
            outer: function() { /* ... */ }
        },
        /* ... */
    },
}
```

outer 함수가 실행되면 outer 함수의 LexicalEnvironment 객체가 생성됩니다.
outer 함수의 LexicalEnvironment 객체의 OutterEnviromentReference는 전역 LexicalEnvironment를 참조합니다.
inner 함수가 실행되기 전까지의 outer 함수의 LexicalEnvironment 객체는 다음과 같습니다.

```javascript
LexicalEnvironment = {
    EnviromentRecord: {
        DeclartionEnvironmentRecord = {
            inner: function() { /* ... */ }
        },
        /* ... */
    },
    OutterEnviromentReference: global
}
```

inner 함수의 LexicalEnvironment 객체는 다음과 같습니다.

```javascript
LexicalEnvironment = {
    EnviromentRecord: {
        DeclartionEnvironmentRecord = {
        },
        /* ... */
    },
    OutterEnviromentReference: outer
}
```

식별자 a가 존재하지 않습니다만 console.log(a)는 정상적으로 값을 출력합니다.
현재 LexicalEnvironment 객체 내에 식별자가 존재하지 않는 경우 OutterEnviromentReference를 참조하기 때문입니다.
참조를 위해 타고 올라간(?) outer의 LexicalEnvironment 객체 내에도 a가 존재하지 않습니다.
이번에는 outer의 OutterEnviromentReference인 전역 환경의 LexicalEnvironment 객체를 참조합니다.
a가 존재하므로 에러를 출력하지 않고 정상적으로 console.log(a)를 출력할 수 있습니다.
이처럼 OutterEnviromentReference는 연결리스트 형태를 띠며, 오직 자신이 *선언된 시점의 LexicalEnvironment*만 참조하고 있으므로
가장 가까운 요소부터 차례대로만 접근 가능하고, 다른 순서로 접근하는 것은 불가능합니다.
이런 구조적 특성 덕분에 여러 스코프에서 동일한 식별자를 선언한 경우, 무조건 스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근 가능합니다.
예제를 통해 살펴보겠습니다.

```javascript
var a = 1;

var outer = function() {
    var inner = function() {
        console.log(a); // undefined

        var a = 3;
    };

    inner();
    console.log(a); // 1
};

outer();
console.log(a); // 1
```

먼저 outer 함수가 실행되기 전까지의 전역의 LexicalEnvironment를 살펴보겠습니다.
전역 환경이므로 OutterEnviromentReference가 존재하지 않습니다.

```javascript
LexicalEnvironment = {
    EnviromentRecord: {
        DeclartionEnvironmentRecord = {
            a: 1,
            outer: function() { /* ... */ }
        },
        /* ... */
    },
}
```

outer 함수가 실행되면 outer 함수의 LexicalEnvironment 객체가 생성됩니다.
outer 함수의 LexicalEnvironment 객체의 OutterEnviromentReference는 전역 LexicalEnvironment를 참조합니다.
inner 함수가 실행되기 전까지의 outer 함수의 LexicalEnvironment 객체는 다음과 같습니다.

```javascript
LexicalEnvironment = {
    EnviromentRecord: {
        DeclartionEnvironmentRecord = {
            inner: function() { /* ... */ }
        },
        /* ... */
    },
    OutterEnviromentReference: global
}
```

console.log(a)가 실행되기 전까지의 inner 함수의 LexicalEnvironment 객체는 다음과 같습니다.

```javascript
LexicalEnvironment = {
    EnviromentRecord: {
        DeclartionEnvironmentRecord = {
            a: undefined
        },
        /* ... */
    },
    OutterEnviromentReference: outer
}
```

var 키워드를 선언된 식별자는 등록과 동시에 undefined로 초기화됩니다.
이 상태에서 console.log(a)를 실행하면 현재의 LexicalEnvironment에 이미 a라는 식별자가 존재하므로 OutterEnviromentReference를 참조하지 않고 undefined를 출력합니다.
**무조건 스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근 가능하다**는 말이 바로 이런 뜻입니다.
inner 함수 실행이 종료되고 outer 함수가 console.log(a)를 실행하면 현재 outer 함수의 LexicalEnvironment 객체 내에 a가 존재하지 않으므로
OutterEnviromentReference 참조해 상위 LexicalEnvironment에서 a가 발견될 때까지 스코프 체인을 타고 a를 검색합니다.
전역 LexicalEnvironment의 내부에 a가 있으므로 정상적으로 값을 출력합니다.
전역의 console.log(a) 또한 전역 자신의 LexicalEnvironment 내부에 a가 존재하므로 정상적으로 값을 출력합니다.
위 코드 상의 식별자 a는 전역 공간에서도 선언했고, inner 함수 내부에서도 선언했습니다.
때문에 inner 함수 내부에서 a에 접근하려고 하면 무조건 스코프 체인 상의 첫 번째 인자, 즉 inner 스코프의 LexicalEnvironment부터 검색할 수밖에 없습니다.
inner 스코프 LexicalEnvironment에 식별자 a가 존재하므로 스코프 체인 검색을 더 진행하지 않고 즉시 현재 스코프 상의 a를 반환하게 됩니다.
즉 inner 함수 내부에서 a 변수를 선언했기 때문에 전역 공간에서 선언한 동일한 이름의 a 변수에는 접근할 수 없는 셈입니다.
이를 *변수 은닉화*라고 합니다.

####

## 💬 마치며

실행 컨텍스트는 실행할 코드에 제공할 환경 정보들을 모아 놓은 객체이며,
전역 공간에서 자동으로 생성되는 전역 컨텍스트와 eval 및 함수 실행에 의한 컨텍스트가 있는 것을 지난 파트에서 살펴봤습니다.
실행 컨텍스트 객체 자체를 이해하고 나면 스코프, 스코프 체인은 자연스럽게 이해되는 개념입니다.
