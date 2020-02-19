# 프로토타입 체인

## 🤘 목표

-   [ ]

####

## 🖼️ 메서드 오버라이드

prototype 객체를 참조하는 \_\_proto\_\_를 생략하면 인스턴스는 prototype에 정의된 프로퍼티나 메서드를 마치 자신의 것처럼 사용할 수 있다고 지난 파트에서 설명했습니다.
그런데 인스턴스가 동일한 이름의 프로퍼티 또는 메서드를 가지고 있는 상황을 가정해 봅시다.
코드를 통해 살펴보겠습니다.

```javascript
const Person = function(name) {
    this.name = name;
};

Person.prototype.getName = function() {
    return this.name;
};

const iu = new Person('지금');

iu.getName = function() {
    return `바로 ${this.name}`;
};

console.log(iu.getName()); // 바로 지금
```

iu.\_\_proto\_\_.getName이 아닌 iu 객체에 있는 getName 메서드가 호출됐습니다.
이 같은 현상을 메서드 오버라이드라고 합니다.
원본을 제거하고 다른 대상으로 교체하는 것이 아니라, 원본이 그대로 있는 상태에서 다른 대상을 그 위에 얹는 이미지를 떠올리면 정확합니다.

자바스크립트 엔진이 getName이라는 메서드를 찾는 방식은 가장 가까운 대상인 자신의 프로퍼티를 검색하고, 없으면 그 다음으로 가까운 대상인 \_\_proto\_\_를 검색하는 순서로 진행됩니다.
그러니까 \_\_proto\_\_에 있는 메서드는 자신에게 있는 메서드보다 검색 순서에서 밀려 호출되지 않은 것뿐입니다.
앞서 *교체*가 아니라 *얹는다*고 표현했듯, 두 가지는 구분할 필요가 있습니다.
교체하는 형태라면 원본에는 접근할 수 없지만, 얹는 형태라면 원본이 아래에 유지되고 있으므로 접근할 수 있는 방법이 있습니다.
메서드 오버라이딩이 이뤄진 상태에서 prototype에 있는 메서드에 접근하는 방법을 코드를 통해 살펴보겠습니다.

```javascript
console.log(iu.__proto__.getName()); // undefined
```

this가 prototype 객체(iu.\_\_proto\_\_)를 가리키는데 prototype 상에는 name 프로퍼티가 없기 때문에 undefined가 출력됐습니다.

```javascript
Person.prototype.name = '이지금';

console.log(iu.__proto__.getName()); // 이지금
```

원하는 메서드(prototype에 있는 getName)가 호출되고 있다는 점이 확실해졌으나, this가 prototype을 바라보고 있기 때문에 이것을 인스턴스를 바라보도록 수정하겠습니다.

```javascript
console.log(iu.__proto__.getName.call(iu)); // 지금
```

위와 같이 오버라이딩이 이뤄진 상태에서 prototype에 있는 메서드에 접근할 수 있습니다.

####

## ⛓️ 프로토타입 체인

프로토타입 체인을 설명하기에 앞서 개발자 도구를 이용해 객체 내부 구조를 살펴보겠습니다.

```javascript
console.dir({ a: 1 });
```

첫 줄을 통해 Object의 인스턴스임을 알 수 있고, 프로퍼티 a의 값 1이 보이고, \_\_proto\_\_ 내부에는 hasOwnproperty, isPrototypeOf, toLocalString, toString, valueOf 등의 메서드가 보입니다.
constructor는 생성자 함수인 Object를 가리키고 있습니다.

```javascript
console.dir([1, 2]);
```

배열 리터럴의 \_\_proto\_\_에는 pop, push 등의 익숙한 배열 메서드 및 constructor가 존재합니다.
그런데 이 \_\_proto\_\_ 안에는 또다시 \_\_proto\_\_가 등장합니다.
열어보면 객체의 \_\_proto\_\_와 동일한 내용으로 이뤄져있습니다.
prototype 객체가 *객체*이기 때문입니다.
기본적으로 모든 객체의 \_\_proto\_\_에는 Object.prototype이 연결됩니다.
prototype도 예외가 아닙니다.

앞서 \_\_proto\_\_는 생략 가능하다고 했습니다.
그렇기 때문에 Array.prototype 내부의 메서드를 마치 인스턴스 자신의 것처럼 실행할 수 있습니다.
생략 가능한 \_\_proto\_\_를 한번 더 따라가면 Object.prototype을 참조할 수 있기 때문입니다.
어떤 데이터의 \_\_proto\_\_ 프로퍼티 내부에 다시 \_\_proto\_\_ 프로퍼티가 연쇄적으로 이어진 것을 프로토타입 체이닝이라고 합니다.
프로토타입 체이닝은 앞서 소개한 메서드 오버라이드와 동일한 맥락입니다.

어떤 메서드를 호출하면 자바스크립트 엔진은 데이터 자신의 프로퍼티를 검색해서 원하는 메서드가 있으면 그 메서드를 실행하고,
없으면 \_\_proto\_\_를 검색해서 있으면 그 메서드를 실행하고, 없으면 다시 \_\_proto\_\_를 검색해서 실행하는 식으로 진행합니다.
배열에서 배열 메서드 및 객체 메서드를 실행하는 과정을 예제 코드를 통해 살펴보면 다음과 같습니다.

```javascript
const arr = [1, 2];

arr.push(3); // [1, 2, 3]
/* 실제로는 아래와 같이 체이닝되는 셈입니다 */
arr(.__proto__).push(3);

arr.hasOwnproperty(2); // true
/* 실제로는 아래와 같이 체이닝되는 셈입니다 */
arr(.__proto__)(.__proto__).hasOwnproperty(2)
```

예제를 더 살펴보겠습니다.

```javascript
const arr = [1, 2];

Array.prototype.toString.call(arr); // 1, 2
Object.prototype.toString.call(arr); // [object Array]
arr.toString(); // 1, 2

arr.toString = function() {
    return this.join('_');
};

arr.toString(); // 1_2
```

arr 변수는 배열이므로 arr.\_\_proto\_\_는 Array.prototype을 참조하고, Array.prototype은 객체이므로 Array.prototype.\_\_proto\_\_는 Object.prototype을 참조합니다.
toString이라는 이름을 가진 메서드는 Array.prototype뿐만 아니라 Object.prototype에도 있습니다.
때문에 어디에 바인딩해 호출하느냐에 따라 결과값이 다른 것입니다.
또한, arr에 toString이라는 메서드를 직접 설정해주면, 메서드 오버라이드와 프로토타입 체이닝에 의해 arr의 메서드인 toString이 바로 실행됩니다.

#### 객체 전용 메서드의 예외사항

어떤 생성자 함수든 prototype은 반드시 객체이기 때문에 Object.prototype이 언제나 프로토타입 체인의 최상단에 존재하게 됩니다.
따라서 객체에서만 사용할 메서드는 다른 여느 데이터 타입처럼 프로토타입 객체 안에 정의할 수 없습니다.
객체에서만 사용할 메서드를 Object.prototype 내부에 정의한다면 다른 데이터 타입도 해당 메서드를 사용할 수 있기 때문입니다.

```javascript
Object.prototype.getEntries = function() {
    const res = [];

    for (var prop in this) {
        if (this.hasOwnProperty(prop)) {
            res.push([prop, this[prop]]);
        }
    }

    return res;
};

const data = [
    ['object', { a: 1, b: 2, c: 3 }], // [["a", 1], ["b", 2], ["c", 3]]
    ['number', 345], // []
    ['string', 'abc'], // [["0", "a"], ["1", "b"], ["2", "c"]]
    ['boolean', false], // []
    ['func', function() {}], // []
    ['array', [1, 2, 3, 4, 5]], // [["0", "1"], ["1", "2"], ["2", "3"], ["3", "4"], ["4", "5"]]
];

data.forEach((datum) => {
    console.log(datum[1].getEntries());
});
```

원래 의도대로라면 객체가 아닌 다른 데이터 타입에서는 오류를 던져야 하지만, 모든 데이터가 오류 없이 결과를 반홥합니다.
어느 데이터 타입이든 프로토타입 체이닝을 통해 getEntrie 메서드에 접근할 수 있습니다.
이같은 이유로 객체만을 대상으로 동작하는 객체 전용 메서드들은 부득이 Object.prototype이 아닌 Object에 스태틱 메서드로 부여할 수밖에 없습니다.

또한 생성자 함수인 Object와 인스턴스인 객체 리터럴 사이에는 this를 통한 연결이 불가능하기 때문에 여느 전용 메서드처럼 *메서드명 앞의 대상이 곧 this*가 되는 방식 대신 this 사용을 포기하고
대상 인스턴스를 인자로 직정 주입하는 방식으로 구현돼 있습니다.
만약 객체 전용 메서드에 대해서도 다른 데이터 타입과 마찬가지 규칙을 적용할 수 있었다면, Object.freeze(instance)의 경우 instance.freeze()와 같은 방식으로 표현할 수 있었을 것입니다.
즉, 객체 한정 메서드들을 Object.prototype이 아닌 Object에 직접 부여할 수밖에 없는 까닭은 Object.prototype이 \_\_proto\_\_에 반복 접근하므로써 도달할 수 있는 최상위 존재이기 때문입니다.

*프로토타입 체인상 가장 마지막에는 언제나 Object.prototype이 있다*는 명제에 예외적인 경우도 있습니다.
Object.create를 이용하는 경우인데요, Object.create(null)는 \_\_proto\_\_가 없는 객체를 생성합니다.
예제 코드를 살펴 보겠습니다.

```javascript
const _proto = Object.create(null);

_proto.getValue = function(key) {
    return this[key];
};

const obj = Object.create(_proto);

obj['a'] = 1;

console.log(obj.getValue('a')); // 1;
console.dir(obj);
```

\_proto에는 \_\_proto\_\_ 프로퍼티가 없는 객체를 할당했습니다.
obj는 \_proto를 \_\_proto\_\_로 하는 객체를 할당했습니다.
이제 obj를 출력해보면, \_\_proto\_\_에는 오직 getValue 메서드만 존재하고, \_\_proto\_\_ 및 constructor 프로퍼티 등은 존재하지 않습니다.
이런 방식으로 만든 객체는 일반적인 데이터에 반드시 존재하는 빌트인 메서드 및 프로퍼티가 제거됨으로써 기본 기능에 제약이 생긴 대신, 객체 자체가 가벼워지는 성능상 이점이 있습니다.

#### 다중 프로토타입 체인

자바스크립트의 기본 내장 데이터 타입들은 모두 프로토타입 체인이 1단계(객체)이거나 2단계(나머지)로 끝나는 경우만 있었지만 사용자가 새롭게 만드는 경우에는 그 이상도 얼마든지 가능합니다.
예제 코드를 통해 살펴보겠습니다.

```javascript
const Grade = function() {
    const args = Array.prototype.slice.call(arguments);
    console.log(this);

    for (let i = 0; i < args.length; i++) {
        this[i] = args[i];
    }

    this.length = args.length;
};

const g = new Grade(100, 80);
```

상수 g는 Grade의 인스턴스를 바라봅니다.
Grade의 인스턴스는 여러개의 인자를 받아 각각 순서대로 인덱싱해서 저장하고 length 프로퍼티가 존재하는 등 배열의 형태를 지니지만,
배열의 메서드를 사용할 수는 없는 유사배열 객체입니다.
인스턴스에서 배열 메서드를 직접 쓰게 만들기 위해서는 다음과 같이 g.\_\_proto\_\_, 즉 Grade.prototype이 배열의 인스턴스를 바라보게 하면 됩니다.

```javascript
Grade.prototype = [];

console.log(g); // Grade(2) [100, 80]
```

이제 인스턴스 g는 프로토타입 체인에 따라 g 객체 자신이 지니는 멤버, Grade의 prototype에 있는 멤버, Array.prototype에 있는 멤버, 마지막으로 Object.prototype에 있는 멤버까지 접근(3단계)할 수 있습니다.

####

## 💬 마치며

이번 절에서는 메서드 오버라이드와 프로토타입 체인, 다중 프로토타입 체인까지 살펴봤습니다.
다중 프로토타입 체인에 관해서는 **두 단계 이상의 체인을 지니는 다중 프로토타입 체인**도 가능하다는 사실을 확인하고, 다음 장인 class 파트에서 조금 더 심도 있게 다루도록 하겠습니다.
