# 프로토타입 개념

## 🤘 목표

-   [ ] 프로토타입을 이해한다.

####

## 🐣 프로토타입

자바스크립트는 프로토타입 기반 언어입니다.
클래스 기반 언어에서는 **상속(확장)**을 사용하지만, 프로토타입 기반 언어에서는 어떤 객체를 **원형**으로 삼고 이를 *복제(참조)*함으로써 *상속(확장)*과 비슷한 효과를 얻습니다.
유명한 프로그래밍 언어의 상당수가 클래스 기반인 것에 비교하면 프로토타입은 꽤나 독특한 개념이라 할 수 있습니다.

####

## 🐥 프로토타입 개념

```javascript
const instance = new Constructor();
```

1. 어떤 생성자 함수(Constructor)를 new 연산자와 함께 호출합니다. ← 위 예제의 경우 new Constructor();
2. Constructor에서 정의된 내용을 바탕으로 새로운 instance가 생성됩니다. ← 위 예제의 경우 식별자 instance
3. 이때 instance에는 **\_\_proto\_\_**라는 프로퍼티가 자동으로 부여됩니다.
4. 이 프로퍼티는 Constructor의 prototype 프로퍼티를 참조합니다.

~~그다지 직관적이라는 생각이 들지 않아~~ 책의 도식은 따로 이미지를 첨부하지 않았습니다. 🤔

prototype이라는 프로퍼티와 \_\_proto\_\_라는 프로퍼티가 새로 등장했는데, 이 둘의 관계가 프로토타입 개념의 핵심입니다.
prototype은 객체이며, 이를 참조하는 \_\_proto\_\_ 역시 객체입니다.
prototype 객체 내부에는 인스턴스가 사용할 메서드를 저장합니다.
그러면 인스턴스에서도 숨겨진 프로퍼티인 \_\_proto\_\_를 통해 이 메서드들에 접근할 수 있게 됩니다.
참고로 \_\_proto\_\_를 발음할 때는 *던더 프로토*라고 합니다.
dunder는 double underscore의 줄임말이라고 합니다.

ES5.1 명세에는 \_\_proto\_\_가 아니라 [[prototype]]이라는 명칭으로 정의돼 있습니다.
\_\_proto\_\_라는 프로퍼티는 사실 브라우저가 [[prototype]]을 구현한 대상에 지나지 않았습니다.
명세에는 instance.\_\_proto\_\_와 같은 방식으로 직접 접근하는 것은 허용되지 않고 오직 Object.getPrototypeOf(instance) 또는 Reflect.getPrototypeOf(instance)를 통해서만 접근할 수 있도록 정의했습니다.
그러나 이런 명세에도 불구하고 대부분의 브라우저가 \_\_proto\_\_에 직접 접근하는 방식을 포기하지 않았고,
결국 ES6에서는 이를 브라우저에서 동작하는 레거시 코드에 대한 호환성 유지 차원에서 정식으로 인정하기 시작했습니다.

다만 어디까지나 브라우저에서의 호환성을 고려한 지원일 뿐, 권장되는 방식은 아니며 브라우저가 아닌 다른 환경에서는 얼마든지 이 방식이 지원되지 않을 수 있습니다.
그러므로 이 글에서는 이해의 편의를 위해 \_\_proto\_\_를 사용합니다만, 학습 목적으로만 이해하고 실무에서는 가급적 \_\_proto\_\_를 사용하지 않기를 권장합니다.
[Object.getPrototypeOf()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf), [Object.create()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/create), [Reflect.getPrototypeOf()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/getPrototypeOf) 등을 이용하도록 합니다.
_기본적으로 \_\_proto\_\_는 생략 가능합니다._

####

## 🐤 constructor

생성자 함수의 프로퍼티인 prototype 객체 내부에는 constructor라는 프로퍼티가 있습니다.
인스턴스의 \_\_proto\_\_ 객체 내부에도 마찬가지입니다.
이 프로퍼티는 단어 그대로 원래의 생성자 함수(자기 자신)를 참조합니다.
자신을 참조하는 프로퍼티를 굳이 왜 가지고 있을까 싶지만, 이 역시 인스턴스와의 관계에 있어서 필요한 정보입니다.
인스턴스로부터 그 원형이 무엇인가를 알 수 있는 수단이기 때문입니다.

```javascript
const arr = [1, 2];

Array.prototype.constructor === Array; // true
arr.__proto__.constructor === Array; // true
arr.constructor === Array; // true (기본적으로 __proto__는 생략 가능합니다!)

const arr2 = new arr.constructor(3, 4);

console.log(arr2); // [3, 4]
```

개발자 도구를 통해 하나하나 직접 살펴보면 이해하기 어렵지 않습니다.
arr2와 같이 constructor는 읽기 전용 속성이 부여된 예외적인 경우(기본형 리터럴 변수인 number, string, boolean)을 제외하고는 값을 바꿀 수 있습니다.
아래 예제를 통해 살펴보겠습니다.

```javascript
const NewConstructor = function() {
    console.log('this is new constructor!');
};

const dataTypes = [
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

모든 데이터가 d instanceof NewConstructor 명령에 대해 false를 반환합니다.
constructor를 변경하더라도 참조하는 대상이 변경될 뿐, 이미 만들어진 인스턴스의 원형이 바뀐다거나 데이터 타입이 변하는 것은 아님을 알 수 있습니다.
때문에 어떤 인스턴스의 생성자 정보를 알아내기 위해 constructor 프로퍼티에 의존하는 것은 안전하지 않습니다.
비록 어떤 인스턴스로부터 생성자 정보를 알아내는 유일한 수단인 constructor가 항상 안전하지는 않지만 오히려 그렇기 때문에 클래스 상속을 _흉내내는_ 것이 가능해진 측면도 있겠습니다.
이에 대해서는 class 파트에서 다루도록 하겠습니다.

마지막으로 constructor에 접근하는 방법에 대한 예제를 살펴보고 prototype과 instance로 넘어가도록 하겠습니다.

```javascript
const Person = function(name) {
    this.name = name;
};

const p1 = new Person('사람1'); // { name: '사람1' }, true
const p1Proto = Object.getPrototypeOf(p1);
const p2 = new Person.prototype.constructor('사람2'); // { name: '사람2' }, true
const p3 = new p1Proto.constructor('사람3'); // { name: '사람3' }, true
const p4 = new p1.__proto__.constructor('사람4'); // { name: '사람4' }, true
const p5 = new p1.constructor('사람5'); // { name: '사람5' }, true

[p1, p2, p3, p4, p5].forEach((p) => {
    console.log(p, p instanceof Person);
});
```

p1 ~ p5는 모두 Person의 인스턴스입니다.

####

## 🐟 prototype, instance

```javascript
const Person = function(name) {
    // 화살표 함수로 작성할 수 없습니다!
    this._name = name;
};

Person.prototype.getName = function() {
    return this._name;
};
```

예제코드를 자세히 살펴보기 전에 앞서 주석을 살펴보겠습니다.
주석에 *화살표 함수로 작성할 수 없다*고 적어뒀습니다.
화살표 함수는 생성자 함수로 사용할 수 없습니다.
생성자 함수는 prototype 프로퍼티를 가지며, prototype 프로퍼티가 가리키는 prototype 객체의 constructor를 사용합니다.
하지만 화살표 함수는 prototype 프로퍼티를 가지고 있지 않습니다.
또한 화살표 함수 내부에는 this가 존재하지 않고, 접근하려고 하면 스코프체인상 가장 가까운 this에 접근하게 됩니다.
이 때문에 만일 getName 메서드를 화살표 함수로 작성할 경우, 상위 스코프인 window를 참조할 것이므로 마찬가지로 undefined를 반환합니다.

```javascript
const Foo = () => {};

// 화살표 함수는 prototype 프로퍼티가 없다
console.log(Foo.hasOwnProperty('prototype')); // false

const foo = new Foo(); // TypeError: Foo is not a constructor
```

Person이라는 생성자 함수의 prototype에 getName이라는 메서드를 지정했습니다.
이제 Person의 인스턴스는 \_\_proto\_\_ 프로퍼티를 통해 getName을 호출할 수 있습니다.
아래 예제를 통해 살펴보겠습니다.

```javascript
const soulcactus = new Person('soulcactus');

Person.prototype === soulcactus.__proto__; // true
soulcactus.__proto__.getName(); // undefined
```

instance인 soulcactus의 \_\_proto\_\_가 Constructor인 Person()의 prototype 프로퍼티를 참조하므로 둘은 같은 객체를 바라봅니다.(true)

그런데 메서드 호출의 결과로 undefined가 나왔습니다.
'soulcactus'란 값이 나오지 않는 것보다 **에러가 발생하지 않았다**는 점에 초점을 맞춰 보면,
어떤 함수를 실행해 undefined가 나왔다는 것은 이 함수가 *호출할 수 있는 함수*에 해당한다는 것을 의미합니다.
만약 함수가 아닌 다른 데이터 타입이었다면 TypeError가 발생했을 것입니다.
그런데 에러가 아닌 값, 즉 undefined가 나왔으므로 getName()이 실제로 실행됐음을 알 수 있고, 또한 getName이 함수라는 것이 입증된 셈입니다.

그렇다면 함수 내부에서 왜 undefined를 반환하는지 살펴보겠습니다.
getName은 this.name을 리턴하는 함수입니다.
어떤 함수를 _메서드로서_ 호출할 때 메서드명 바로 앞의 객체가 곧 this가 된다고 했습니다.
soulcactus.\_\_proto\_\_.getName()에서 getName 함수 내부에서의 this는 soulcactus가 아니라 soulcactus.\_\_proto\_\_라는 객체가 된 것입니다.
객체 내부에 name 프로퍼티가 없으므로 '찾고자 하는 식별자가 정의돼 있지 않을 때는 Error 대신 undefined를 반환한다'는 자바스크립트 규약에 의해 undefined가 반환된 것입니다.

만일 \_\_proto\_\_ 객체에 name 프로퍼티가 있다면 어떨까요?

```javascript
const soulcactus = new Person('soulcactus');

soulcactus.__proto__._name = 'SOULCACTUS';
soulcactus.__proto__.getName(); // SOULCACTUS
```

예상대로 잘 출력되며, 관건은 this라는 것을 알 수 있습니다.
앞서 언급한 것처럼 기본적으로 \_\_proto\_\_는 생략 가능합니다.
때문에 \_\_proto\_\_를 생략하고 인스턴스에서 곧바로 메서드를 호출하면 메서드의 this가 자연스럽게 인스턴스를 가리킵니다.

```javascript
const soulcactus = new Person('soulcactus');

soulcactus.getName(); // soulcactus
```

다른 예제를 통해 더 살펴보도록 하겠습니다.

```javascript
const Constructor = function(name) {
    this.name = name;
};

Constructor.prototype.method1 = function() {};
Constructor.prototype.property1 = 'Constructor Prototype Property';

const instance = new Constructor('instance');

console.dir(Constructor);
console.dir(instance);
```

직접 개발자 도구를 통해 살펴보겠습니다. 생성자인 Constructor의 prototype을 인스턴스인 instance의 \_\_proto\_\_가 그대로 참조하고 있는 것을 볼 수 있습니다.
이번에는 대표적인 내장 생성자 함수인 Array를 바탕으로 다음 예제 코드를 살펴보도록 하겠습니다.

```javascript
const arr = [1, 2];

console.dir(arr);
console.dir(Array);
```

이번에도 직접 개발자 도구를 열어 먼저 arr를 살펴보면, 첫 줄에 Array(2)가 표기되어 있습니다.
Array라는 생성자 함수를 원형으로 삼아 생성됐고, length가 2임을 알 수 있습니다.
\_\_proto\_\_를 열어보면 Array 생성자 함수가 가지고 있는 prototype를 참조하고 있습니다.
배열뿐만 아니라 null, undefined를 제외한 모든 데이터 타입에는 그에 대응하는 생성자 함수가 있습니다.(String 등)

new 연산자를 이용한 방식과 그렇지 않은 방식에 대해서는 이 파트의 범위를 넘어서는 것이므로 관련 링크를 첨부하도록 하겠습니다.

-   [왜 new Array()보다 []의 선호도가 높을까?](https://withhsunny.tistory.com/71)
-   [문자열 원형과 String 객체의 차이](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/String)

무리한 표현이지만, 프로토타입 개념을 인간의 언어의 빗대어 말하면
어떤 제작자(constructor)가 어떤 모양(prototype 내부의 메서드 등의 기능)을 닮은 붕어빵틀(prototype)을 제공하면 그 틀을 빌려(참조) 붕어빵(instance)을 찍어내는 것과 같습니다.
붕어빵틀(prototype)과 꼭 닮은 붕어빵(instance)은 그 틀(prototype)과 제작자(constructor)를 기억합니다.
이미 구워진 빵처럼(?) 특별한 성질(타입)의 붕어빵(number, string, boolean)이 아니라
떡반죽같은 성질(타입)의 붕어빵(number, string, boolean 외 타입)이라면(?) 제작자(constructor)는 변경 가능한데요,
그렇다고 해서 붕어빵이라는 붕어빵 본연의 성질(타입)인 밀가루라는 사실까지 바뀌는 것은 아닙니다.

####

## 💬 마치며

클래스에 익숙한 많은 개발자들이 자바스크립트를 배척하는 이유로 프로토타입이 어렵고 복잡하다는 점을 들지만,
오히려 자바스크립트는 프로토타입 개념을 제대로 이해하는 것만으로도 이미 숙련자 레벨에 도달할 수 있는 시야를 확보하게 되는 셈이므로 두려워할 일은 아니라고 생각합니다.
알고 나면 의외로 매우 쉬운 개념이기도 합니다.
확실히 숙지하고 나면 숙련자 레벨에 도달할 수 있을 것입니다.
