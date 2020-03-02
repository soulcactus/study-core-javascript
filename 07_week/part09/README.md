# 프로토타입 개념

## 🧐 프로토타입

자바스크립트는 프로토타입 기반 언어입니다. 클래스 기반 언어에서는 *상속*을 사용하지만 프로토타입 기반 언어에서는 어떤 객체를 *원형(prototype)으로 삼고 이를 복제(참조)함으로써 상속과 비슷한 효과*를 얻습니다. 유명한 프로그래밍 언어의 상당수가 클래스 기반인 것에 비교하면 프로토타입은 꽤나 독특한 개념이라 할 수 있습니다.

## 🧐 프로토타입의 개념 이해

```javascript
var instance = new Constructor();
```

위 코드의 흐름을 살펴보겠습니다.

1. 어떤 생성자 함수(Constructor)를 new 연산자와 함께 호출하면

2. Constructor에서 정의된 내용을 바탕으로 새로운 인스턴스(instance)가 생성됩니다.

3. 이때 instance에는 __proto__라는 프로퍼티가 자동으로 부여되는데

4. 이 프로퍼티는 Constructor의 prototype이라는 프로퍼티를 참조합니다.

prototype이라는 프로퍼티와 \_\_proto\_\_라는 프로퍼티가 새로 등장했는데요. 이 둘의 관계가 프로토타입 개념의 핵심이라고 할 수 있습니다.

prototype은 객체이며 이를 참조하는 \_\_proto\_\_ 역시 당연히 객체입니다. prototype 객체 내부에는 인스턴스가 사용할 메서드를 저장합니다. 그러면 인스턴스에서도 숨겨진 프로퍼티인 \_\_proto\_\_를 통해 이 메서드들에 접근할 수 있게됩니다.

* \_\_proto\_\_를 읽을 때는 'dunder proto', '던더 프로토'하고 발음하면 된다고 하며 여기서 dunder는 'double underscore'의 줄임말이라고 합니다.

ES5.1 명세에는 \_\_proto\_\_가 아니라  [[prototype]]이라는 명칭으로 정의돼 있습니다. \_\_proto\_\_라는 프로퍼티는 사실 브라우저들이 [[prototype]]을 구현한 대상에 지나지 않습니다.
명세에는 또 instance.\_\_proto\_\_와 같은 방식으로 직접 접근하는 것은 허용하지 않고 오직 Object.getPrototypeOf(instance) / Refelect.getPrototypeOf(instance)를 통해서만 접근할 수 있도록 정의했었는데요. 그러나 이런 명세에도 불구하고 대부분의 브라우저들이 \_\_proto\_\_에 직접 접근하는 방식을 포기하지 않았습니다. 결국 ES6에서는 이를 브라우저에서 동작하는 레거시 코드에 대한 호환성 유지 차원에서 정식으로 인정하기에 이르렀습니다. 다만, 어디까지나 브라우저에서의 호환성을 고려한 지원일 뿐 권장되는 방식은 아니며, 브라우저가 아닌 다른 환경에서는 얼마든지 이 방식이 지원되지 않을 가능성이 있습니다.

그러므로 이 글에서는 이해의 편의를 위해 \_\_proto\_\_를 사용하지만, 학습 목적으로만 이해하고 실무에서는 가급적 \_\_proto\_\_를 사용하지 않기를 권장합니다. 대신 **Object.getPrototypeOf() / Object.create()** 등을 이용할 수 있습니다.

## 🧐 prototype, instance

```javascript
var Person = function(name) {
    this._name = name;
};

Person.prototype.getName = function() {
    return this._name;
};
```

Person이라는 생성자 함수의 prototype에 getName이라는 메서드를 지정했습니다.
이제 Person의 인스턴스는 \_\_proto\_\_ 프로퍼티를 통해 getName을 호출할 수 있습니다.

그 이유는 instance의 \_\_proto\_\_가 Construcor의 prototype 프로퍼티를 참조하므로 결국 둘은 같은 객체를 바라보기 때문입니다.

```javascript
var hwizzzang = new Person("hwizzzang");
hwizzzang.__proto__.getName(); // undefined

Person.prototype === suzi.__proto__ // true
```

위 예제에서 메서드 호출 결과로 undefined가 나왔는데요. 'hwizzzang' 이라는 값이 나오지 않은 것보다는 *에러가 발생하지 않았다*는 점이 우선입니다. 어떤 변수를 실행해 undefined가 나왔다는 것은 이 변수가 *호출할 수 있는 함수*에 해당한다는 것을 의미합니다. 만약 실행할 수 없는, 즉 함수가 아닌 다른 데이터 타입이었다면 TypeError가 발생했을 것입니다. 위 예제에서는 에러가 아닌 다른 값이 나왔으므로 getName이 실제로 실행됐음을 알 수 있고, 이로부터 getName이 함수라는 것을 알 수 있습니다.

그런데 왜 함수 내부에서 undefined를 반환했을까요?

getName은 this._name을 반환하도록 구성되어있습니다.
어떤 함수를 *메서드로서* 호출할 때는 메서드명 바로 앞의 객체가 곧 this가 된다고 했었는데요. hwizzzang.\_\_proto\_\_.getName() 에서 getName 함수 내부에서의 this는 hwizzzang이 아니라 hwizzzang.\_\_proto\_\_라는 객체가 됩니다. 이 객체 내부에는 name 프로퍼티가 없으므로 *찾고자 하는 식별자가 정의돼 있지 않을 경우에는 Error 대신 undefined를 반환한다*라는 자바스크립트 규약에 의해 undefined가 반환된 것입니다.

그럼 만약 \_\_proto\_\_ 객체에 name 프로퍼티가 있다면 어떤 결과가 나올까요?

``` javascript
var hwizzzang = new Person("hwizzzang");
hwizzzang.__proto__.name = "hwizzzang__proto__";
hwizzzang.__proto__.getName(); // hwizzzang__proto__
```

예상대로 hwizzzang\_\_proto\_\_가 잘 출력되며, 관건은 this라는 것을 알 수 있습니다.

this를 인스턴스로 하는 방법은 \_\_proto\_\_ 없이 인스턴스에서 곧바로 메서드를 쓰면 됩니다.

```javascript
var hwizzzang = new Person("hwizzzang", 24);
hwizzzang.getName(); // hwizzzang
var soulcactus = new Person("soulcactus", 31);
soulcactus.getName(); // soulcactus
```

\_\_proto\_\_는 생략 가능한 프로퍼티입니다. 원래부터 생략 가능하도록 정의돼있었는데요. 이 정의 를 바탕으로 자바스크립트의 전체 구조가 구성됐다고 해도 과언이 아닙니다. *생략 가능한 프로퍼티*라는 개념은 이해의 영역이 아니므로 \_\_proto\_\_가 생략 가능하다는 점만 기억하면 됩니다.

```javascript
hwizzzang.__proto__.getName
-> hwizzzang(.__proto__).getName
-> hwizzzang.getName
```

\_\_proto\_\_를 생략하지 않으면 this는 hwizzzang.\_\_proto\_\_를 가리키지만, 이를 생략하면 hwizzzang을 가리킵니다. hwizzzang.\_\_proto\_\_에 있는 메서드인 getName을 실행하지만 this는 hwizzzang을 바라보게 할 수 있게 된 것입니다.

여기서 프로토타입의 개념을 좀 더 상세히 설명하자면 이렇습니다.

자바스크립트는 함수에 자동으로 객체인 prototype 프로퍼티를 생성해 놓습니다. new 연산자와 함께 함수를 호출할 경우(생성자 함수로서 사용할 경우)에는 그로부터 생성된 인스턴스에는 숨겨진 프로퍼티인 \_\_proto\_\_가 자동으로 생성되며, 이 프로퍼티는 생성자 함수의 prototype 프로퍼티를 참조합니다. \_\_proto\_\_ 프로퍼티는 생략 가능하도록 구현돼 있기 때문에 **생성자 함수의 prototype에 어떤 메서드나 프로퍼티가 있다면 인스턴스에서도 마치 자신의 것처럼 해당 메서드나 프로퍼티에 접근할 수 있게 됩니다.**

밑에서 다른 예제를 통해 더 살펴보겠습니다.

```javascript
var Constructor = function(name) {
    this.name = name;
};

Constructor.prototype.method1 = function() {};
Constructor.prototype.property1 = "Constructor Prototype Property";

var instance = new Constructor("instance");

console.dir(Constructor);
console.dir(instance);
```

console.dir(Constructor);를 실행하면 Constructor의 디렉터리 구조가 출력됩니다.
출력 결과의 첫 줄에는 함수라는 의미인 f, 함수 이름인 Constructor, 인자인 name이 보입니다. 그 내부에는 여러 프로퍼티가 보이는데,  여기서 다시 prototype 프로퍼티를 열어보면 추가한 method1, property1, constructor, \_\_proto\_\_ 등이 보입니다.

* 출력 결과를 살펴볼 때 글자 색상에서 차이가 나는데, 이런 색상의 차이는 { enumerable: false } 속성이 부여된 프로퍼티인지의 여부에 따릅니다. 짙은색은 열거 가능한 프로퍼티임(enumerable)을 의미하고, 옅은색은 열거할 수 없는 프로퍼티임(innumerable)을 의미합니다. for in 등으로 객체의 프로퍼티 전체에 접근하고자 할 때 접근 가능 여부를 색상으로 구분지어 표기하는 것입니다.

console.dir(instance); 코드는 instance의 디렉터리 구조를 출력하라고 했습니다. 그런데 출력 결과로는 Constructor가 나옵니다. 어떤 생성자 함수의 인스턴스는 해당 생성자 함수의 이름을 표기함으로써 해당 함수의 인스턴스임을 표기하고 있습니다. 이 출력 결과를 열어보면 Constructor의 prototype과 동일한 내용으로 구성돼 있음을 확인할 수 있습니다.

밑에서는 대표적인 내장 생성자 함수인 Array를 바탕으로 살펴보겠습니다.

```javascript
var arr = [1, 2];
console.dir(arr);
console.dir(Array);
```

arr 변수를 출력한 결과를 살펴보겠습니다. Array(2)를 보면 Array라는 생성자 함수를 원형으로 삼아 생성됐고, length가 2임을 알 수 있습니다. \_\_proto\_\_를 열어보면 배열에 사용하는 메서드들이 모두 들어있습니다.

Array를 출력한 결과도 살펴보면 함수라는 의미의 f가 표시돼 있고, 이후 줄부터는 함수의 기본적이 프로퍼티들인 arguments, caller, length, name 등이 보입니다. 더해서 Array 함수의 정적 메서드인 from, isArray, of 등도 있습니다. prototype을 열어보면 arr 변수를 출력했을 때의 \_\_proto\_\_와 완전히 동일한 내용으로 구성돼 있음을 확인할 수 있습니다.

Array를 new 연산자와 함께 호출해서 인스턴스를 생성하든, 그냥 배열 리터럴을 생성하든 instance인 [1, 2]가 만들어집니다. 이 인스턴스의 \_\_proto\_\_는 Array.prototype을 참조하는데, \_\_proto\_\_가 생략 가능하도록 설계돼 있기 때문에 인스턴스가 push, pop, forEach 등의 메서드를 마치 자신의 것처럼 호출할 수 있습니다.

```javascript
var arr = [1, 2];
arr.forEach(function() {}); // (O)
Array.isArray(arr); // (O) true
arr.isArray(); // (X) TypeError: arr.isArray is not a function
```

위의 코드를 살펴보겠습니다. Array의 prototype 프로퍼티 내부에 있지 않은 from, isArray 등의 메서드들은 인스턴스가 직접 호출할 수 없으므로 Array 생성자 함수에서 직접 접근해야 실행 가능합니다.

## 🧐 Constructor 프로퍼티

생성자 함수의 프로퍼티인 prototype 객체 내부에는 constructor라는 프로퍼티가 있습니다. 인스턴스의 \_\_proto\_\_ 객체 내부에도 마찬가지인데요. 이 프로퍼티는 단어 그대로 **원래의 생성자 함수(자기 자신)을 참조**합니다. 이 프로퍼티는 인스턴스로부터 그 원형이 무엇인지를 알 수 있는 수단이 됩니다.

```javascript
var arr = [1, 2];
Array.prototype.constructor === Array // true
arr.__proto__.constructor === Array // true
arr.constructor === Array // true

var arr2 = new arr.constructor(3, 4);
console.log(arr2); // [3, 4]
```

인스턴스의 \_\_proto\_\_가 생성자 함수의 prototype 프로퍼티를 참조하며 \_\_proto\_\_가 생략가능하기 때문에 인스턴스에서 직접 constructor에 접근할 수 있는 수단이 생깁니다. 이와 같은 이유로 6번째 줄과 같은 명령도 오류 없이 동작합니다.

constructor는 읽기 전용 속성이 부여된 예외적인 경우(number, string, boolean)를 제외하고는 값을 바꿀 수 있습니다.

```javascript
var NewConstructor = function() {
    console.log('this is new constructor!');
};

var dataTypes = [
    1, // Number & false
    'test', // String & false
    true, // Boolean & false
    {}, // NewConstructor & false
    [], // NewConstructor & false
    function() {}, // NewConstructor & false
    /test/, // NewConstructor & false
    new Number(), // NewConstructor & false
    new String(), // NewConstructor & false
    new Boolean(), // NewConstructor & false
    new Object(), // NewConstructor & false
    new Function(), // NewConstructor & false
    new RegExp(), // NewConstructor & false
    new Date(), // NewConstructor & false
    new Error(), // NewConstructor & false
];

dataTypes.forEach((d) => {
    d.constructor = NewConstructor;
    console.log(d.constructor.name, '&', d instanceof NewConstructor);
});
```

모든 데이터가 d instanceof NewConstructor 명령에 대해 false를 반환합니다. 이를 통해 constructor를 변경하더라도 참조하는 대상이 변경될 뿐 이미 만들어진 인스턴스의 원형이 바뀐다거나 데이터 타입이 변하는 것은 아님을 알 수 있습니다. 어떤 인스턴스의 생성자 정보를 알아내기 위해 constructor 프로퍼티에 의존하는 게 항상 안전하지는 않은 것입니다.

비록 어떤 인스턴스로부터 생성자 정보를 알아내는 유일한 수단인 constructor가 항상 안전하지는 않지만 오히려 그렇기 때문에 클래스 상속을 흉내내는 것이 가능해진 측면도 있습니다. 이에 대해서는 class 파트에서 자세히 다루겠습니다.

정리차원에서 다양한 constructor에 접근하는 방법에 대한 예제를 살펴보겠습니다.

```javascript
var Person = function(name) {
    this.name = name;
};

var p1 = new Person("사람1"); // { name: "사람1" }, true
var p1Proto = Object.getPrototypeOf(p1);
var p2 = new Person.prototype.constructor("사람2"); // { name: "사람2" }, true
var p3 = new p1Proto.constructor("사람3"); // { name: "사람3" }, true
var p4 = new p1.__proto__.constructor("사람4"); // { name: "사람4" }, true
var p5 = new p1.constructor("사람5"); // { name: "사람5" }, true

[p1, p2, p3, p4, p5].forEach((p) => {
    console.log(p, p instanceof Person);
});
```

p1부터 p5까지는 모두 Person의 인스턴스입니다.


## 정리

어떤 생성자 함수를 new 연산자와 함께 호출하면 Contructor에서 정의된 내용을 바탕으로 새로운 인스턴스가 생성되는데, 이 인스턴스에는 \_\_proto\_\_라는 Contructor의 prototype 프로퍼티를 참조하는 프로퍼티가 자동으로 부여됩니다. \_\_proto\_\_는 생략 가능한 속성이라서 인스턴스는 Contructor.prototype의 메서드를 마치 자신의 메서드인 것처럼 호출할 수 있습니다.

Constructor.prototype에는 contructor라는 프로퍼티가 있는데, 이는 다시 생성자 함수 자신을 가리킵니다. 이 프로퍼티는 인스턴스가 자신의 생성자 함수가 무엇인지를 알고자 할 때 필요한 수단입니다.