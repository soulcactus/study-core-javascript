# this

다른 대부분의 객체지향 언어에서 this는 클래스로 생성한 인스턴스 객체를 의미하며 클래스에서만 사용할 수 있기 때문에 혼란의 여지가 없거나 많지 않습니다.

그러나 자바스크립트에서의 this는 어디서든 사용할 수 있습니다. 상황에 따라 바라보는 대상이 달라지는데, 어떤 이유로 그렇게 되는지를 파악하기 힘든 경우도 있고 예상과 다르게 엉뚱한 대상을 바라보는 경우도 있습니다.

이번 장에서는 상황별로 this가 어떻게 달라지는지, 왜 그렇게 되는지 알아보고 예상과 다른 대상을 바라보고 있을 경우 그 원인을 효과적으로 추적하는 방법 등을 살펴보겠습니다.

## **🧐상황에 따라 달라지는 this**

자바스크립트에서 this는 기본적으로 실행 컨텍스트가 생성될 때 함께 결정됩니다. 실행 컨텍스트는 함수를 호출할 때 생성되므로, 바꿔 말하면 this는 **함수를 호출할 때 결정**된다고 할 수 있겠습니다. 함수를 어떤 방식으로 호출하느냐에 따라 값이 달라지는 것입니다.

지금부터 각 상황별로 this가 어떤 값을 보게 되는지, 그 원인은 무엇인지 알아보겠습니다.

### **1. 전역 공간에서의 this**

전역 공간에서의 this는 **전역 객체**를 가리킵니다. 개념상 전역 컨텍스트를 생성하는 주체가 바로 전역 객체이기 때문입니다. 전역 객체는 자바스크립트 런타임 환경에 따라 다른 이름과 정보를 가지고 있습니다. 브라우저 환경에서 전역객체는 window이고 Node.js 환경에서는 global입니다.

이번 장의 주제인 this와 큰 관련은 없지만 전역 공간을 다루는 김에 잠시 전역 공간에서만 발생하는 특이한 성질 하나를 살펴보고 넘어가겠습니다.

전역변수를 선언하면 자바스크립트 엔진은 이를 전역객체의 프로퍼티로도 할당합니다. 변수이면서 객체의 프로퍼티이기도 한 셈입니니다.

```javascript
var a = 1;
console.log(a); // 1
console.log(window.a); // 1
console.log(this.a); // 1
```

전역공간에서 선언한 변수 a에 1을 할당했을 뿐인데 window.a 와 this.a 모두 1이 출력됩니다. 이러한 값이 출력되는 이유는 **자바스크립트의 모든 변수는 특정 객체의 프로퍼티로서 동작**하기 때문입니다. 사용자가 var 연산자를 이용해 변수를 선언하더라도 실제 자바스크립트 엔진은 어떤 특정 객체의 프로퍼티로 인식하는 것입니다.

여기서 특정 객체란, 앞서 배운 실행 컨텍스트의 LexicalEnvironment(이하 L.E)입니다. 실행 컨텍스트는 변수를 수집하여 L.E의 프로퍼티로 저장하며, 이후 어떤 변수를 호출하면 L.E를 조회하여 일치하는 프로퍼티가 있을 경우 그 값을 반환합니다. 전역 컨텍스트의 경우 L.E는 전역객체를 그대로 참조합니다.

a를 직접 호출할 때도 1이 나오는 까닭은 무엇일까요? 이는 변수 a에 접근하고자 할 때 스코프 체인에서 a를 검색하다가 가장 마지막에 도달하는 전역 스코프의 L.E, 즉 전역 객체에서 해당 프로퍼티 a를 발견하여 그 값을 반환하기 때문입니다.

더해서, 전역 공간에서는 var로 변수를 선언하는 대신 window의 프로퍼티에 직접 할당하더라도 결과적으로 var로 선언한 것과 똑같이 동작할 것이라는 예상을 할 수 있을 것입니다. 대부분의 경우에는 이 말이 맞습니다.

```javascript
var a = 1;
window.b = 2;
console.log(a, window.a, this.a); // 1, 1, 1
console.log(b, window.b, this.b); // 2, 2, 2

window.a = 3;
b = 4;
console.log(a, window.a, this.a); // 3, 3, 3
console.log(b, window.b, this.b); // 4, 4, 4
```

그러나 전역변수 선언과 전역객체의 프로퍼티 할당 사이에 선혀 다른 경우도 있습니다. 바로 **삭제** 명령에 대해 그렇습니다.

```javascript
var a = 1;
delete window.a; // false
console.log(a, window.a, this.a); // 1 1 1

var b = 2;
delete b; //false
console.log(b, window.b this.b); // 2 2 2

window.c = 3;
delete window.c; // true
console.log(c, window.c this.c);
// Uncaught ReferenceError: c is not defined

window.d = 4;
delete window.d; // true
console.log(d, window.d this.d);
// Uncaught ReferenceError: d is not defined
```

변수에 delete 연산자를 쓰는 것이 이상해보일 수도 있는데, 앞서 설명한 바와 같이 (window.)를 생략한 것으로 이해하면 됩니다. 전역변수가 곧 전역객체의 프로퍼티이므로 문제가 되지 않습니다.

예제를 살펴보면 처음부터 전역객체의 프로퍼티로 할당한 경우에는 삭제가 되는 반면, 전역변수로 선언한 경우에는 삭제가 되지 않는 것을 확인할 수 있습니다. 즉, 전역변수를 선언하면 자바스크립트 엔진이 이를 자동으로 전역객체의 프로퍼티로 할당하면서 추가적으로 해당 프로퍼티의 configurable 속성(변경 및 삭제 가능성)을 false로 정의하는 것입니다.

이처럼 var로 선언한 전역변수와 전역객체의 프로퍼티는 호이스팅 여부 및 configurable 여부에서 차이를 보입니다.

### **2. 메서드로서 호출할 때 그 메서드 내부에서의 this**

#### **함수 vs 메서드**

어떤 함수를 실행하는 방법은 여러 가지가 있는데, 가장 일반적인 방법에는 함수로서 호출하는 경우, 메서드로서 호출하는 경우 이 두 가지가 있습니다. 이 둘을 두분하는 유일한 차이는 **독립성**에 있습니다.

함수는 그 자체로 *독립적인 기능을 수행*하는 반면, 메서드는 *자신을 호출한 대상 객체에 관한 동작을 수행*합니다. 자바스크립트는 상황별로 this 키워드에 다른 값을 부여하게 함으로써 이를 구현했습니다.

간혹 메서드를 '객체의 프로퍼티에 할당된 함수'로 이해하시는 분들이 있는데, 반은 맞고 반은 틀립니다. 어떤 함수를 객체의 프로퍼티에 할당한다고 해서 그 자체로서 무조건 메서드가 되는 것이 아니라 **객체의 메서드로서 호출할 경우에만 메서드로 동작하고, 그렇지 않으면 함수로 동작**합니다. 밑의 예제를 보겠습니다.

```javascript
var func = function(x) {
    console.log(this, x);
};
func(1); // window { ... } 1

var obj = {
    method : func
};
obj.method(2); // { method: ƒ } 2
obj['method'](1); // { method: ƒ } 1 // 대괄호 표기법
```

func라는 변수에 익명함수를 할당하고 func를 호출했더니 this로 전역객체 window가 출력됩니다. 다음은 obj라는 변수에 객체를 할당하는데, 그 객체의 method 프로퍼티에 앞에서 만든 func 함수를 할당했습니다. 이후 obj의 method를 호출했더니, 이번에는 this가 obj라고 합니다.

obj의 method 프로퍼티에 할당한 값과 func 변수에 할당한 값은 모두 1번째 줄에서 선언한 함수를 참조합니다. 즉, 원래의 익명함수는 그대로인데 이를 변수에 담아 호출한 경우와 obj 객체의 프로퍼티에 할당해서 호출한 경우에 this가 달라지는 것입니다.

'함수로서의 호출'과 '메서드로서의 호출'을 구분하는 방법은 **점**(.)을 확인하시면 됩니다. func(1)은 앞에 점이 없으니 함수로서 호출한 것이고, obj.method(2);는 앞에 점이 있으니 메서드로서 호출한 것입니다. 대괄호 표기법에 따른 경우에도 메서드로서 호출한 것입니다.

다시 말해 점 표기법이든 대괄호 표기법이든, 어떤 함수를 호출할 때 그 함수 이름(프로퍼티명) 앞에 객체가 명시돼 있는 경우에는 메서드로 호출한 것이고, 그렇지 않은 모든 경우에는 함수로 호출한 것입니다.

#### **메서드 내부에서의 this**

this에는 호출한 주체에 대한 정보가 담깁니다. 어떤 함수를 메서드로서 호출하는 경우 호출 주체는 바로 함수명(프로퍼티명) 앞의 객체입니다. 점 표기법의 경우 마지막 점 앞에 명시된 객체가 곧 this가 되는 것입니다.

```javascript
var obj = {
    methodA : function() { console.log(this); },
    inner: {
        methodB : function() { console.log(this); }
    }
};

obj.methodA(); // { methodA : f, inner: { ... } }    ( === obj )
obj['mehtodA'](); // { methodA : f, inner: { ... } }    ( === obj )

obj.inner.methodB(); // { methodB : f }    ( === obj.inner )
obj.inner['methodB'](); // { methodB : f }    ( === obj.inner )
obj['inner'].methodB(); // { methodB : f }    ( === obj.inner )
obj['inner']['methodB'](); // { methodB : f }    ( === obj.inner )
```

### **3. 함수로서 호출할 때 그 함수 내부에서의 this**

#### **함수 내부에서의 this**

어떤 함수를 함수로서 호출할 경우에는 this가 지정되지 않습니다. this에는 호출한 주체에 대한 정보가 담긴다고 했었는데요. 그러나 함수로서 호출하는 것은 호출 주체(객체지향언어에서의 객체)를 명시하지 않고 개발자가 코드에 직접 관여해서 실행한 것이기 때문에 호출 주체의 정보를 알 수 없습니다.

지난시간에 실행 컨텍스트를 활성화할 당시에 this가 지정되지 않은 경우 this는 전역 객체를 바라본다고 했습니다. 따라서 함수에서의 this는 전역 객체를 가리킵니다. 더글라스 크락포드는 이를 명백한 설계상의 오류라고 지적하는데요. 그 이유는 바로 이어서 설명하겠습니다.

#### **메서드의 내부함수에서의 this**

메서드 내부에서 정의하고 실행한 함수에서의 this는 자바스크립트 초심자들이 this에 관해 가장 자주 혼란을 느끼는 지점 중 하나입니다. 그러나 우리는 이미 어떤 함수를 메서드로서 호출할 때와 함수로서 호출할 때 this가 무엇을 가리키는지를 알고 있습니다. 내부함수 역시 이를 함수로서 호출했는지, 메서드로서 호출했는지만 파악하면 this의 값을 정확히 맞출 수 있습니다.

다음 예제의 각 console.log 위치에서 this가 무엇을 가리키는지 예상해보고 정답과 비교해보세요.

```javascript
var obj1 = {
    outer : function() {
        console.log(this); // (1)

        var innerFunc = function() {
            console.log(this); // (2) (3)
        }

        innerFunc();

        var obj2 = {
            innerMethod: innerFunc
        };

        obj2.innerMethod();
    }
};

obj1.outer();

// (1) : { outer: ƒ }
// (2) : window { ... }
// (3) : { innerMethod: f }
```

1. 마지막 줄에서 obj1.outer를 호출합니다. 이 함수는 호출할 때 함수명인 outer 앞에 점(.)이 있었으므로 메서드로서 호출한 것입니다. 따라서 this에는 마지막 점 앞의 객체인 obj1이 바인딩되고 (1) 에서는 obj1의 객체 정보가 출력됩니다.

2. innerFunc()를 호출할 때 이 함수명 앞에 점(.)이 없었습니다. 즉 함수로서 호출한 것이므로 this가 지정되지 않았고, 자동으로 스코프 체인상 최상위 객체읜 전역객체(window)가 바인딩되며 window 객체 정보가 출력됩니다.

3. obj2.innerMethod()를 호출할 때 함수명인 innerMethod 앞에 점(.)이 있었으므로 메소드로서 호출한 것입니다. 따라서 this에는 마지막 점 앞의 객체인 obj2가 바인딩되며 obj2객체 정보가 출력됩니다.

this 바인딩에 관해서는 함수를 실행하는 당시의 주변 환경(메서드 내부인지, 함수 내부인지 등)은 중요하지 않고, 오직 **해당 함수를 호출하는 구문 앞에 점 또는 대괄호 표기가 있는지 없는지**가 관건입니다.

이렇게 하면 this에 대한 구분은 명확히 할 수 있지만, 그 결과 this라는 단어가 주는 인상과는 사뭇 달라져 버렸습니다. 호출 주체가 없을 때는 자동으로 전역객체를 바인딩하지 않고 호출 당시 주변 환경의 this를 그대로 상속받아 사용할 수 있다면 좋겠네요. 변수를 검색하면 우선 가장 가까운 스코프의 L.E를 찾고 없으면 상위 스코프를 탐색하듯이, this 역시 현재 컨텍스트에 바인딩된 대상이 없으면 직전 컨텍스트의 this를 바라보도록 말입니다.

#### **메서드의 내부 함수에서의 this를 우회하는 방법**

아쉽게도 ES5까지는 자체적으로 내부함수에 this를 상속할 벙법이 없지만 다행히 이를 우회할 방법이 없지는 않습니다. 그중 대표적인 방법은 바로 *변수를 활용*하는 것입니다.

```javascript
var obj = {
    outer : function() {
        console.log(this); // { outer : f }

        var innerFunc1 = function() {
            console.log(this); // window { ... }
        }

        innerFunc1();

        var self = this;

        var innerFunc2 = function() {
            console.log(self); // { outer : f }
        };
        innerFunc2();

    }
};

obj.outer();
```

위 예제의 innerFunc1 내부에서 this는 전역객체를 가리킵니다. 한편 outer 스코프에서 self라는 변수에 this를 저장한 상태에서 호출한 innerFunc2의 경우 self에는 객체 obj가 출력됩니다.

#### **this를 바인딩하지 않는 함수**

ES6에서는 함수 내부에서 this가 전역객체를 바라보는 문제를 보완하고자, this를 바인딩하지 않는 화살표 함수를 새로 도입했습니다. 화살표 함수는 실행 컨텍스트를 생성할 때 this 바인딩 과정 자체가 빠지게 되어, 상위 스코프의 this를 그대로 활용할 수 있습니다. 내부함수를 화살표 함수로 바꾸면 이전에 알아본 '우회법'이 불필요해집니다.

```javascript
var obj = {
    outer : function() {
        console.log(this); // (1) { outer: f }

        var innerFunc = () => {
            console.log(this); // (2) { outer: f }
        };

        innerFunc();

    }
};

obj.outer();
```

그 밖에도 call, aplly 등의 메서드를 활용해 함수를 호출할 때 명시적으로 this를 지정하는 방법이 있습니다. 이러한 방법에 대해서는 뒤에서 소개하겠습니다.

### **4. 콜백 함수 호출 시 그 함수 내부에서의 this**

함수 A의 제어권을 다른 함수(또는 메서드) B에게 넘겨주는 경우 함수 A를 콜백 함수라 합니다. 이때 함수 A는 함수 B의 내부 로직에 따라 실행되며, this 역시 함수 B 내부 로직에서 정한 규칙에 따라 값이 결정됩니다.

콜백 함수도 함수이기 때문에 기본적으로 이전에 알아본 것과 마찬가지로 this가 전역객체를 참조하지만, 제어권을 받은 함수에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조하게 됩니다.

다음은 대표적인 콜백 함수입니다. 각각 어떤 값이 출력되는지를 예측해보고 결과와 비교해보시길 바랍니다.

```javascript
setTimeout(function() { console.log(this); }, 300); // (1) 0.3초 뒤 전역 객체 출력

[1, 2, 3, 4, 5].forEach(function (x) { // (2)
    console.log(this, x); // 전역객체와 배열의 각 요소 총 5회 출력
});

document.body.innerHTML += "<button id="a">클릭</button>";// (3)
document.body.querySelector("#a").addEventListener("click", function(e) {
    console.log(this, e); // 버튼 클릭 시 지정한 엘리먼트와 클릭 이벤트에 관한 정보가 담긴 객체 출력
})
```

1. (1) : setTimeout 함수는 300ms 만큼 시간 지연을 한 뒤 콜백 함수를 실행하라는 명령입니다. 0.3초 뒤 전역 객체가 출력됩니다.

2. (2) : forEach 메서드는 배열의 각 요소를 앞에서부터 차례로 하나씩 꺼내어 그 값을 콜백함수의 첫 번쨰 인자로 삼아 함수를 실행하라는 명령입니다. 전역객체와 배열의 각 요소가 총 5회 출력됩니다.

3. (3) : addEventListener는 지정한 엘리먼트에 'click'이벤트가 발생할 때마다 그 이벤트 정보를 콜백 함수의 첫 번째 인자로 삼아 함수를 실행하라는 명령입니다. 버튼을 클릭하면 앞서 지정한 엘리먼트와 클릭 이벤트에 관한 정보가 담긴 객체가 출력됩니다.

(1)의 setTimeout 함수와 (2)의 forEach 메서드는 그 내부에서 콜백 함수를 호출할 때 대상이 될 this를 지정하지 않습니다. 따라서 콜백 함수 내부에서의 this는 전역객체를 참조합니다.

한편, (3)의 addEventListener 메서드는 콜백 함수를 호출할 때 자신의 this를 상속하도록 정의돼 있습니다. 그러므로 메서드명의 점(.) 앞부분이 곧 this가 됩니다.

이처럼 콜백 함수에서의 this는 '무조건 이거다!'라고 정의할 수 없습니다. 콜백 함수의 제어권을 가지는 함수(메서드)가 콜백 함수에서의 this를 무엇으로 할지를 결정하며, 특별히 정의하지 않은 경우에는 기본적으로 함수와 마찬가지로 전역객체를 바라봅니다.

### **5. 생성자 함수 내부에서의 this**

생성자 함수는 어떤 공통된 성질을 지니는 객체들을 생성하는 데 사용하는 함수입니다. 객체지향 언어에서는 생성자를 클래스(class), 클래스를 통해 만든 객체를 인스턴스(instance)라고 합니다.

프로그래밍적으로 '생성자'는 구체적인 인스턴스를 만들기 위한 일종의 틀입니다. 이 틀에는 해당 클래스의 공통 속성들이 미리 준비돼 있고, 여기에 구체적인 인스턴스의 개성을 더해 개별 인스턴스를 만들 수 있는 것입니다.

자바스크립트는 함수에 생성자로서의 역할을 함께 부여했습니다. new 명령어와 함께 함수를 호출하면 해당 함수가 생성자로서 동작하게 됩니다. 그리고 어떤 함수가 생성자 함수로서 호출된 경우 내부에서의 this는 곧 새로 만들 구체적인 인스턴스 자신이 됩니다.

생성자 함수를 호출하면 우선 생성자의 prototype 프로퍼티를 참조하는 __proto__라는 프로퍼티가 있는 객체(인스턴스)를 만들고, 미리 준비된 공통 속성 및 개성을 해당 객체(this)에 부여합니다. 이렇게해서 구체적인 인스턴스가 만들어집니다. 밑 예제에서 확인해보겠습니다.

```javascript
var Cat = function(name, age) {
    this.bark = "야옹";
    this.name = name;
    this.age = age;
};

var choco = new Cat("초코", 7);
var nabi = new Cal("나비", 5);
console.log(choco, nabi);
```

Cat이란 변수에 익명 함수를 할당하고 내부에서 this에 접근하여 bark, name, age 프로퍼티에 각각 값을 대입합니다. 이후 new 명령어와 함께 Cat 함수를 호출하여 변수 choco, nabi에 각각 할당합니다. choco와 nabi를 출력해보면 각각 Cat 클래스의 인스턴스 객체가 출력됩니다. 즉, 각 생성자 함수 내부에서의 this는 choco, nabi 인스턴스를 가리킴을 알 수 있습니다.

## **🧐명시적으로 this를 바인딩하는 방법**

### **1. call 메서드**

```javascript
Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])
```

call 메서드는 메서드의 호출 주체인 함수를 즉시 실행하도록 하는 명령입니다. call 메서드의 첫 번째 인자를 this로 바인딩하고, 이후의 인자들을 호출할 함수의 매개변수로 합니다. 함수를 그냥 실행하면 this는 전역객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있습니다.

```javascript
var func = function(a, b, c) {
    console.log(this, a, b, c);
};

func(1, 2, 3); // window{ ... } 1 2 3
func.call({ x : 1 }, 4, 5, 6); // { x : 1 } 4 5 6
```

메서드에 대해서도 마찬가지로 객체의 메서드를 그냥 호출하면 this는 객체를 참조하지만 call 메서드를 이용하면 임의의 객체를 this로 지정할 수 있습니다.

```javascript
var obj = {
    a: 1,
    method: function(x, y) {
        console.log(this.a, x, y);
    }
};

obj.method(2, 3); // 1 2 3
obj.method.call({ a: 4 }, 5, 6); // 4 5 6
```

### **2. apply 메서드**

```javascript
Function.prototype.aplly(thisArg[, argsArray])
```

apply 메서드는 call 메서드와 기능적으로 완전히 동일합니다.

call 메서드는 첫 번째 인자를 제외한 나머지 모든 인자들을 호출할 함수의 매개변수로 지정하는 반면, apply 메서드는 두 번째 인자를 배열로 받아 그 배열의 요소들을 호출할 함수의 매개변수로 지정한다는 점에서만 차이가 있습니다.

```javascript
var func = function(a, b, c) {
    console.log(this, a, b, c);
}

func.apply({ x: 1 }, [4, 5, 6]); // { x: 1 } 4 5 6

var obj = {
    a: 1,
    method: function(x, y) {
        console.log(this.a, x, y);
    }
};

obj.method.apply({ a: 4 }, [5, 6]) // 4 5 6
```

### **3. call / apply 메서드의 활용**

#### **유사배열객체(array-like object)에 배열 메서드를 적용**

```javascript
var obj = {
    0: "a",
    1: "b",
    2: "c",
    length: 3
};

Array.prototype.push.call(obj, "d"); // 배열 메서드 push를 객체 obj에 적용
console.log(obj); // { 0: "a", 1: "b", 2: "c", 3: "d", length: 4 }

var arr = Array.prototype.slice.call(obj); // slice 메서드를 적용해 객체를 배열로 전환
console.log(arr); // ["a", "b", "c", "d"]
```

객체에는 배열 메서드를 직접 적용할 수 없습니다. 그러나 **키가 0 또는 양의 정수인 프로퍼티가 존재하고 length 프로퍼티의 값이 0 또는 양의 정수**인 객체, 즉 배열의 구조와 유사한 객체의 경우(유사배열객체) call 또는 applyd 메서드를 이용해 배열 메서드를 차용할 수 있습니다.

* slice 메서드는 원래 시작 인덱스값과 마지막 인덱스값을 받아 시작값부터 마지막값의 앞부분까지 배열 요소를 추출하는 메서드인데, 매개변수를 아무것도 넘기지 않을 경우에는 그냥 원본 배열의 얕은 복사본을 반환합니다. 위의 예제에서는 call 메서드를 이용해 원본인 유사배열객체의 얕은 복사를 수행한 것인데, slice 메서드가 배열 메서드이기 때문에 복사본은 배열로 반환하게 된 것입니다.

함수 내부에서 접근할 수 있는 arguments 객체도 유사배열객체이므로 위의 방법으로 배열로 전환해서 활용할 수 있습니다. querySelectorAll, getElementsByClassName 등의 Node 선택자로 선택한 결과인 NodeList도 마찬가지입니다.

```javascript
function a() {
    var argv = Array.prototype.slice.call(arguments);
    argv.forEach(function(arg) {
        console.log(arg);
    });
}

a(1, 2, 3);

document.body.innerHtml = "<div>a</div><div>b</div><div>c</div>";
var nodeList = document.querySelectorAll("div");
var nodeArr = Array.prototype.slice.call(nodeList);
nodeArr.forEach(function(node) {
    console.log(node);
});
```

그 밖에도 유사배열객체에는 call/apply 메서드를 이용해 모든 배열 메서드를 적용할 수 있습니다. 배열처럼 인텍스와 length 프로퍼티를 지니는 문자열에 대해서도 마찬가지입니다.

단, 문자열의 경우 length 프로퍼티가 읽기 전용이기 때문에 원본 문자열에 변경을 가하는 메서드(push, pop, shift, unshift, splice 등)는 에러를 던지며, concat처럼 대상이 반드시 배열이어야 하는 경우에는 에러는 나지 않지만 제대로 된 결과를 얻을 수 없습니다.

```javascript
// call/apply 메서드의 활용 - 문자열에 배열 메서드 적용

var str = "abc def";

// Cannot assign to read only property 'length' of object '[object String]
Array.prototype.push.call(str, ", pushed string");

Array.prototype.concat.call(str, "string"); // [String {"abc def"}, "string"]

Array.prototype.every.call(str, function(char) { // true
    return char !== "";
});

Array.prototype.some.call(str, function(char) { // false
    return char === "";
});

var newArr = Array.prototype.map.call(str, function(char) {
    return char + "!";
});
console.log(newArr); // ["a!", "b!", "c!", " !", "d!", "e!", "f!"]

var newStr = Array.prototype.reduce.apply(str, [
    function(string, char, i ) { // 누산기, 현재 값, 현재 인덱스
        return string + char + i;
    },
    ""
]);
console.log(newStr); // "a0b1c2 3d4e5f6"
```

사실  call/apply를 이용해 형변환하는 것은 'this를 원하는 값으로 지정해서 호출한다'라는 본래 메서드의 의도와는 다소 동떨어진 활용법이라 할 수 있는데요. 이에 ES6에서는 유사배열객체 또는 순회 가능한 모든 종류의 데이터타입을 배열로 전환하는 Array.from 메서드를 새로 도입했습니다.

```javascript
var obj = {
    0: "a",
    1: "b",
    2: "c",
    length: 3
};

var arr = Array.from(obj);
console.log(arr); // ["a", "b", "c"]
```

#### **생성자 내부에서 다른 생성자를 호출**

생성자 내부에 *다른 생성자와 공통된 내용*이 있을 경우 call 또는 apply를 이용해 다른 생성자를 호출하면 간단하게 반복을 줄일 수 있습니다.

```javascript
function Person(name, gender) {
    this.name = name;
    this.gender = gender;
}

function Student(name, gender, school) {
    Person.call(this, name, gender);
    this.school = school;
}

function Employee(name, gender, company) {
    Person.call(this, name, gender);
    this.company = company;
}

var hr = new Student("휘림", "female", "휘쨩대"); // {name: "휘림", gender: "gender", school: "휘쨩대"}
var hj = new Employee("혜진", "female", "(주)소울"); // {name: "휘림", gender: "gender", company: "소울"}
```

#### **여러 인수를 묶어 하나의 배열로 전달하고 싶을 때 - apply**

여러 개의 인수를 받는 메서드에게 하나의 배열로 인수들을 전달하고 싶을 때 apply 메서드를 사용하면 좋습니다. 예를 들어, 배열에서 최대/최솟값을 구해야 할 경우 apply를 사용하지 않는다면 부득이하게 다음과 같은 방식으로 직접 구현할 수밖에 없을 것인데요.

```javascript
var numbers = [10, 20, 3, 16, 45];
var max = min = numbers[0];
numbers.forEach(function(number) {
    if(number > max) {
        max = number;
    }

    if(number < min) {
        min = number;
    }
});

console.log(max, min); // 45 3
```

위의 코드보다는 Math.max / Math.min 메서드에 apply를 적용하면 훨씬 간단해집니다.\

```javascript
var numbers = [10, 20, 3, 16, 45];
var max = Math.max.apply(null, numbers);
var min = Math.min.apply(null, numbers);

console.log(max, min); // 45 3
```

참고로 ES6에서는 펼치기 연산자(spread operator)를 이용하면 apply를 적용하는 것보다 더욱 간편하게 작성할 수 있습니다.

```javascript
const numbers = [10, 20, 3, 15, 45];
const max = Math.max(...numbers);
const min = Math.max(...numbers);

console.log(max, min) // 45 3
```

call/apply 메서드는 명시적으로 별도의 this를 바인딩하면서 함수 또는 메서드를 실행하는 훌륭한 방법이지만 오히려 이로 인해 this를 예측하기 어렵게 만들어 코드 해석을 방해한다는 단점이 있습니다. 그럼에도 ES5 이하의 환경에서는 마땅한 대안이 없기 때문에 실무에서 매우 광범위하게 활용되고 있습니다.

### **4. bind 메서드**

```javascript
Function.prototype.bind(thisArg[, arg1[, agr2[, ...]]])
```

bind 메서드는 ES5에서 추가된 기능으로, call과 비슷하지만 **즉시 호출하지는 않고** 넘겨 받은 this 및 인수들을 바탕으로 **새로운 함수를 반환**하기만 하는 메서드입니다. 다시 새로운 함수를 호출할 때 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달했던 인수들의 *뒤에 이어서 등록*됩니다. 즉, bind 메서드는 함수에 this를 미리 적용하는 것과 부분 적용 함수를 구현하는 두 가지 목적을 모두 지닙니다.

```javascript
var func = function(a, b, c, d, e) {
    console.log(this, a, b, c, d);
};

func(1, 2, 3, 4); // Window{ ... } 1 2 3 4

var bindFunc1 = func.bind({ x: 1 });
bindFunc1(5, 6, 7, 8); // { x: 1 } 5 6 7 8

var bindFunc2 = func.bind({ x: 1 }, 4, 5);
bindFunc2(6, 7); // { x: 1 } 4 5 6 7
bindFunc2(8, 9); // { x: 1 } 4 5 8 9
```

bindFunc1 변수에는 func에 this를 { x: 1 }로 지정한 새로운 함수가 담기며 이후 bindFunc1을 호출하면 원하는 결과인 { x: 1 } 5 6 7 8 를 얻을 수 있습니다. 이 줄의 bind는 this만을 지정한 것입니다.

bindFunc2 변수에는 func에 this를 { x: 1 }로 지정하고, 앞에서부터 두 개의 인수를 각각 4, 5로 지정한 새로운 함수를 담았습니다. 이후 매개변수로 6, 7을 넘기면 this 값이 바뀐 것을 제외하고는 최초 func 함수에 4, 5, 6, 7을 넘긴 것과 같은 동작을 합니다. 다음 줄의 bindFunc2(8, 9);도 마찬가지입니다. 이 줄의 bind는 this 지정과 함께 부분 적용 함수를 구현한 것입니다.

#### **name 프로퍼티**

bind 메서드를 적용하여 새로 만든 함수의 name 프로퍼티에는 동사 bind의 수동태인 'bound'라는 접두어가 붙습니다. 어떤 함수의 name 프로퍼티가 'bound ~~'라면 이는 곧 함수명이 ~~인 원본 함수에 bind 메서드를 적용한 새로운 함수라는 의미가 되므로 기존의 call이나 apply보다 코드를 추적하기에 더 수월해진 면이 있습니다.

```javascript
var func = function(a, b, c, d) {
    console.log(this, a, b, c, d);
};

var bindFunc = func.bind({ x: 1 }, 4, 5);
console.log(func.name) // func
console.log(bundFunc.name) // bound func
```

#### **상위 컨텍스트의this를 내부함수나 콜백함수에 전달하기**

앞에서는 메서드의 내부함수에서 메서드의 this를 그대로 바라보게 하기 위한 방법으로 self 등의 변수를 활용한 우회법을 소개했었는데, call, apply 또는 bind 메서드를 이용하면 더 깔끔하게 처리할 수 있습니다.

```javascript
// 내부함수에서 this 전달 - call
var obj = {
    outer : function() {
        console.log(this); // { outer: f }
        var innerFunc = function() {
            console.log(this); // { outer: f }
        };
        innerFunc.call(this);
    }
};

obj.outer();
```

```javascript
// 내부함수에서 this 전달 - bind
var obj = {
    outer : function() {
        console.log(this); // { outer: f }
        var innerFunc = function() {
            console.log(this); // { outer: f }
        }.bind(this);

        innerFunc();
    }
};

obj.outer();
```

콜백함수를 인자로 받는 함수나 메서드 중에서 기본적으로 콜백 함수 내에서의 this에 관여하는 함수나 메서드에 대해서도 bind 메서드를 이용하면 this 값을 사용자의 입맛에 맞게 바꿀 수도 있습니다.

```javascript
// 내부함수에서 this 전달 - bind
var obj = {
    logThis : function() {
        console.log(this);
    },
    logThisLater1: function() {
        setTimeout(this.logThis, 500);
    },
    logThisLater2: function() {
        setTimeout(this.logThis.bind(this), 1000);
    }
};

obj.logThisLater1(); // Window { ... }
obj.logThisLater2(); // {logThis: ƒ, logThisLater1: ƒ, logThisLater2: ƒ}
```

### **5. 화살표 함수의 예외사항**

ES6에 새롭게 도입된 화살표 함수는 실행 컨텍스트 생성 시 this를 바인딩하는 과정이 제외됐습니다. 즉, 이 함수 내부에는 this가 아예 없으며, 접근하고자 하면 스코프체인상 가장 가까운 this에 접근하게 됩니다.

```javascript
var obj = {
    outer: function() {
        console.log(this); // {outer: ƒ}
        var innerFunc = () => {
            console.log(this); // {outer: ƒ}
        };
        innerFunc();
    }
};

obj.outer();
```

이렇게 하면 별도의 변수로 this를 우회하거나 call/apply/bind를 적용할 필요가 없어 더욱 간결하고 편리합니다.

### **6. 별도의 인자로 this를 받는 경우(콜백 함수 내에서의 this)**

콜백 함수를 인자로 받는 메서드 중 일부는 추가로 this로 지정할 객체(thisArg)를 인자로 지정할 수 있는 경우가 있습니다. 이러한 메서드의 thisArg 값을 지정하면 콜백 함수 내부에서 this 값을 원하는 대로 변경할 수 있습니다.

이런 형태는 여러 내부 요소에 대해 같은 동작을 반복 수행해야 하는 배열 메서드에 많이 포진돼 있으며, 같은 이유로 ES6에서 새로 등장한 Set, Map 등의 메서드에도 일부 존재합니다. 그중 대표적인 배열 메서드인 forEach의 예를 살펴보겠습니다.

```javascript
var report = {
    sum: 0,
    count: 0,
    add: function() {
        var args = Array.prototype.slice.call(arguments);
        args.forEach(function(entry) {
            this.sum += entry;
            ++this.count;
        }, this);
    },

    average: function() {
        return this.sum / this.count;
    }
};

report.add(60, 85, 95);
console.log(report.sum, report.count, report.average()); // 240 3 80
```

add 메서드는 arguments를 배열로 변환하여 args 변수에 담습니다. 다음 줄에서는 이 배열을 순회하면서 콜백 함수를 실행하는데, 이때 콜백 함수 내부에서의 this는 forEach 함수의 두 번째 인자로 전달해준 this가 바인딩됩니다.

average는 sum 프로퍼티를 count 프로퍼티로 나눈 결과를 반환하는 메서드입니다.

report.add(60, 85, 95); 에서 60, 85, 95를 인자로 삼아 add 메서드를 호출하면 이 세 인자를 배열로 만들어 forEach 메서드가 실행됩니다. 콜백 함수 내부의 this는 add 메서드에서의 this가 전달된 상태이므로 add메서드의 this(report)를 그대로 가리키고 있습니다. 따라서 배열을 순회하면서 report.sum, report.count 값이 차례로 바뀌고 순회를 마친 결과 report.sum에는 240, report.count에는 3이 담기게 되어 report.average()의 값으로는 80이 나오게 됩니다.

```javascript
// 콜백 함수와 함꼐  thisArg를 인자로 받는 메서드

Array.prototype.forEach(callback[, thisArg])
Array.prototype.map(callback[, thisArg])
Array.prototype.filter(callback[, thisArg])
Array.prototype.some(callback[, thisArg])
Array.prototype.every(callback[, thisArg])
Array.prototype.find(callback[, thisArg])
Array.prototype.findIndex(callback[, thisArg])
Array.prototype.flatMap(callback[, thisArg])
Array.prototype.from(arrayLike[, thisArg])
Set.prototype.forEach(callback[, thisArg])
Map.prototype.forEach(callback[, thisArg])
```

## **정리**

다음 규칙은 명시적 this 바인딩이 없는 한 늘 성립합니다.

* 전역 공간에서의 this는 전역객체(브라우저에서는 Window, Node.js에서는 global)를 참조합니다.

* 어떤 함수를 메서드로서 호출한 경우 this는 메서드 호출 주체(메서드명 앞의 객체)를 참조합니다.

* 어떤 함수를 함수로서 호출한 경우 this는 전역객체를 참조합니다. 메서드의 내부함수에서도 같습니다.

* 콜백 함수 내부에서의 this는 해당 콜백 함수의 제어권을 넘겨받은 함수가 정의한 바에 따르며, 정의하지 않은 경우에는 전역객체를 참조합니다.

* 생성자 함수에서의 this는 생성될 인스턴스를 참조합니다.

다음은 명시적 this 바인딩입니다. 위 규칙에 부합하지 않은 경우에는 다음 내용을 바탕으로 this를 예측할 수 있습니다.

* call, apply 메서드는 this를 명시적으로 지정하면서 함수 또는 메서드를 호출합니다.

* bind 메서드는 this 및 함수에 넘길 인수를 일부 지정해서 새로운 함수를 만듭니다.

* 요소를 순회하면서 콜백 함수를 반복 호출하는 내용의 일부 메서드는 별도의 인자로 this를 받기도 합니다.