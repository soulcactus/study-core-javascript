# 프로토타입 체인

## **메서드 오버라이드**

prototype 객체를 참조하는 \_\_proto\_\_를 생략하면 인스턴스는 prototype에 정의된 프로퍼티나 메서드를 마치 자신의 것처럼 사용할 수 있다고 했었는데요. 그런데 먄약 인스턴스가 동일한 이름의 프로퍼티 또는 메서드를 가지고 있는 상황이라면 어떨까요?

```javascript
var Person = function(name) {
    this.name = name;
}

Person.prototype.getName = function() {
    return this.name;
}

var iu = new Person("지금");

iu.getName = function() {
    return "바로" + this.name;
};

console.log(iu.getName()); // 바로 지금
```

iu.\_\_proto\_\_.getName이 아닌 iu 객체에 있는 getName 메서드가 호출됐습니다. 여기서 일어난 현상을 메서드 오버라이드라고 합니다. 메서드 위에 메서드를 덮어씌웠다는 표현인데요. 원본을 제거하고 다른 대상으로 교체하는 것이 아닌, 원본이 그대로 있는 상태에서 다른 대상을 그 위에 얹는 이미지를 떠올리면 정확합니다.

자바스크립트 엔진이 getName이라는 메서드를 찾는 방식은 가장 가까운 대상인 자신의 프로퍼티를 검색하고, 없으면 그다음으로 가까운 대상인 \_\_proto\_\_를 검색하는 순서로 진행됩니다.
그러니까 \_\_proto\_\_에 있는 메서드는 자신에게 있는 메서드보다 검색 순서에서 밀려 호출되지 않은 것입니다.

앞에서 '교체'가 아닌 '얹는' 이미지라고 말했는데, 이 둘을 구분할 필요가 있습니다.
교체하는 형태라면 원본에는 접근할 수 없는 형태가 되겠지만, 얹는 형태라면 원본이 아래에 유지되고 있으니 원본에 접근할 수 있는 방법이 있겠죠.

그렇다면 메서드 오버라이딩이 이뤄져 있는 상황에서 prototype에 있는 메서드에 접근하기위해서는 어떻게 하면 될까요?

```javascript
console.log(iu__.proto__.getName()); // undefined
```

this가 prototype 객체(iu.\_\_proto\_\_)를 가리키는데 prototype 상에는 name 프로퍼티가 없기 때문에 undefined가 출력되습니다. 만약 prototype에 name 프로퍼티가 있다면 그 값을 출력할 것입니다.

```javascript
Person.prototype.name = "이지금";
console.log(iu.__proto__.getName()); // 이지금
```

원하는 메서드(prototype에 있는 getName)가 호출되고 있는 게 확실해졌습니다. 다만 this가 prototype을 바라보고 있는데 이것을 인스턴스를 바라보도록 바꿔주면 되겠네요. 이는 call이나 apply로 해결이 가능합니다.

```javascript
console.log(iu.__proto__.getName.call(iu)); // 지금
```

일반적으로 메서드가 오바라이드 된 경우에는 자신으로부터 가장 가까운 메서드에만 접근할 수 있습니다. 그 다음으로 가까운 \_\_proto\_\_의 메서드도 접근이 불가능 한 것은 아닙니다. 위와 같인 방법으로 우회적인 방법을 사용할 수 있습니다.

## **프로토타입 체인**

프로토타입 체인을 설명하기에 앞서 객체와 배열의 내부 구조를 살펴보겠습니다

```javascript
console.dir({ a: 1 });
```

첫 줄을 통해 Object의 인스턴스임을 알 수 있습니다. 프로퍼티 a의 값 1이 보이고, \_\_proto\_\_ 내부에는 hasOwnProperty, isPrototypeOf, toString, valueOf 등의 메서드가 보입니다. constructor는 생성자 함수인 Object를 가리키고 있습니다.

```javascript
console.dir([1, 2]);
```

배열 리터럴의 \_\_proto\_\_에는 pop, push 등의 익숙한 배열 메서드 및 constructor가 있다는 것은 이전 파트에서 확인했었습니다. 추가로, 이 \_\_proto\_\_ 안에는 또다시 \_\_proto\_\_가 등장하는데요. 열어보면 객체의 \_\_proto\_\_와 동일한 내용으로 이뤄져있습니다.
이는 prototype 객체가 '객체'이기 때문입니다. 기본적으로 모든 객체의 \_\_proto\_\_에는 Object.prototype이 연결됩니다. prototype 객체도 예외가 아닙니다.

앞에서 \_\_proto\_\_는 생략 가능하다고 했습니다. 그렇기 때문에 배열이 Array.prototype 내부의 메서드를 마치 자신의 것처럼 실행할 수 있는 것입니다.

마찬가지로 Object.prototype 내부의 메서드도 자신의 것처럼 실행할 수 있습니다. 생략 가능한 \_\_proto\_\_를 한 번 더 따라가면 Object.prototype을 참조할 수 있기 때문입니다.

어떤 데이터의 \_\_proto\_\_ 프로퍼티 내부에 다시 \_\_proto\_\_ 프로퍼티가 연쇄적으로 이어진 것을 프로토타입 체인이라 하고, 이 체인을 따라가며 검색하는 것을 프로포타입 체이닝이라고 합니다.

프로토타입 체이닝은 앞서 소개한 메서드 오버라이드와 동일한 맥락입니다. 어떤 메서드를 호출하면 자바스크립트 엔진은 데이터 자신의 프로퍼티들을 검색하여 원하는 메서드가 있으면 그 메서드를 실행하고, 없으면 \_\_proto\_\_를 검색해서 그 메서드를 실행하는데, 없으면 다시 \_\_proto\_\_를 검색해서 실행하는 식으로 진행합니다.

```javascript
var arr = [1, 2];
Array.prototype.toString.call(arr); // 1, 2
Object.prototype.toString.call(arr); // [Object Array]
arr.toString(); // 1, 2

arr.toString = function() {
    return this.join("_");
};

arr.toString(); // 1_2
```

arr 변수는 배열이므로 arr.\_\_proto\_\_는 Array.prototype을 참조하고, Array.prototype은 객체이므로 Array.prototype.\_\_proto\_\_는 Object.prototype을 참조할 것입니다.

toString이라는 이름을 가진 메서드는 Array.prototype뿐 아니라 object.prototype에도 있습니다. 이 둘 중 어떤 값이 출력되는지를 확인하기 위해 우선 2, 3번째 줄에서 Array, Object의 각 프로토타입에 있는 toString 메서드를 arr에 적용했을 때의 출력값을 미리 확인해 봤습니다.

4번째 줄에서 arr.toString을 실행했더니 Array.prototype.toString을 적용한 것과 결과가 동일합니다.

5번째 줄에서는 arr에 직접 toString 메서드를 부여했습니다. 이제 마지막 줄에서는 Array.prototype.toString이 아닌 arr.toString이 바로 실행될 것입니다.

## **객체 전용 메서드의 예외사항**

어떤 생성자 함수이든 prototype은 반드시 객체이기 때문에 Object.prototype이 언제나 프로토타입 체인의 최상단에 존재하게 됩니다. 따라서 객체에서만 사용할 메서드는 다른 데이터 타입처럼 프로토타입 객체 안에 정의할 수 없습니다. 객체에서만 사용할 메서드를 Object.prototype 내부에 정의한다면 다른 데이터 타입도 해당 메서드를 사용할 수 있게 되기 때문입니다.

```javascript
Object.prototype.getEntries = function() {
    var res = [];
    for (var prop in this) {
        if(this.hasOwnProperty(prop)) {
            res.push([prop, this[prop]]);
        }
    }
    return res;
};

var data = [
    ['object', { a: 1, b: 2, c: 3 }], // [["a", 1], ["b", 2], ["c", 3]]
    ['number', 345], // []
    ['string', 'abc'], // [["0", "a"], ["1", "b"], ["2", "c"]]
    ['boolean', false], // []
    ['func', function() {}], // []
    ['array', [1, 2, 3, 4, 5]], // [["0", "1"], ["1", "2"], ["2", "3"], ["3", "4"], ["4", "5"]]
];

data.forEach(function(datum) {
    console.log(datum[1].getEntries());
});
```

첫 번째 줄에서는 객체에서만 사용할 의도로 getEntries라는 메서드를 만들었습니다. forEach에 따라 각 데이터마다 getEntries를 실행해보니, 모든 데이터가 오류 없이 결과를 반환하고 있습니다.

원래 의도대로라면 객체가 아닌 데이터 타입에 대해서는 오류를 던져야하지만, 어느 데이터 타입이건 프로토타입 체이닝을 통해 getEntries 메서드에 접근할 수 있으므로 오류를 반환하지 않는 것입니다.

이 같은 이유로 객체만을 대상으로 동작하는 객체 전용 메서드들은 부득이 Object.prototype이 아닌 Object에 스태틱 메서드로 부여할 수밖에 없습니다.

또한 생성자 함수인 Object와 인스턴스인 객체 리터럴 사이에는 this를 통한 연결이 불가능하기 때문에 여느 전용 메서드처럼 '메서드명 앞의 대상이 곧 this'가 되는 방식 대신 this의 사용을 포기하고 대상 인스턴스를 인자로 직접 주입해야하는 방식으로 구현돼 있습니다.

만약 객체 전용 메서드에 대해서도 다른 데이터 타입과 마찬가지의 규칙을 적용할 수 있었다면, 예를 들어 Object.freeze(instance)의 경우 instance.freeze()처럼 표현할 수 있었을 것입니다. 즉, instance.\_\_proto\_\_(생성자 함수의 prototype)에 freeze라는 메서드가 있었을 것입니다.

객체 한정 메서드들을 Object.prototype이 아닌 Object에 직접 부여할 수밖에 없던 이유는 Object.prototype이 여타 참조형 데이터뿐 아니라 기본형 데이터조차 \_\_proto\_\_에 반복 접근함으로써 도달할 수 있는 최상위 존재이기 때문입니다.

반대로 같은 이유에서 Object.prototype에는 어떤 데이터에서도 활용할 수 있는 범용적인 메서드들만 있는데요. toString, hasOwnProperty, valueOf, isPrototypeOf 등은 모든 변수가 마치 자신의 메서드인 것처럼 호출할 수 있습니다.

앞서 '프로토타입 체인상 가장 마지막에는 언제나 Object.prototype이 있다'고 했는데, *예외적으로 object.create를 이용하면 Object.prototype의 메서드에 접근할 수 없는 경우*가 있습니다.

Object.create(null)은 \_\_proto\_\_가 없는 객체를 생성합니다.

```javascript
var _proto = Object.create(null);
_proto.getValue = function(key) {
    return this[key];
};

var obj = Object.create(_proto);
obj.a = 1;
console.log(obj.getValue("a")); // 1
console.dir(obj);
```

_proto에는 \_\_proto\_\_ 프로퍼티가 없는 객체를 할당했습니다. 다시 obj는 앞서 만든 _proto를 \_\_proto\_\_로 하는 객체를 할당했습니다.

obj를 출력해보면 \_\_proto\_\_에는 오직 getValue 메서드만이 존재하며, \_\_proto\_\_ 및 consrtuctor 프로퍼티 등은 보이지 않습니다. 이 방식으로 만든 객체는 일반적인 데이터에 반드시 존재하는 내장 메서드 및 프로퍼티들이 제거됨으로써 기본 기능에 제약이 생긴 대신, 객체 자체의 무게가 가벼워짐으로써 성능상 이점을 가집니다.

## **다중 프로토타입 체인**

자바스크립트의 기본 내장 데이터 타입들은 모두 프로토타입 체인이 1단계(객체)이거나 2단계(나머지)로 끝나는 경우만 있었지만 사용자가 새롭게 만드는 경우에는 그 이상도 얼마든지 가능합니다. 다음 예제를 통해 알아보겠습니다.

```javascript
var Grade = function() {
    var args = Array.prototype.slice.call(arguments);
    for (var i = 0; i < args.length; i++) {
        this[i] = args[i];
    }
    this.length = args.length;
};

var g = new Grade(100, 80);
```

변수 g는 Grade ㅇ니스턴스를 바라봅니다. Grade의 인스턴스는 여러 개의 인자를 받아 각각 순서대로 인덱싱하여 저장하고 length 프로퍼티가 존재하는 등 배열의 형태를 지니고 있지만, 배열의 메서드를 사용할 수 없는 유사배열객체입니다. 인스턴스에서 배열 메서드를 직접 쓸 수 있게끔 하기 위해서는 g.\_\_proto\_\_, 즉 Grade.prototype이 배열의 인스턴스를 바라보게 하면 됩니다.

```javascript
Grade.prototype = [];

console.log(g); // Grade(2) [100, 80]
```

이제 인스턴스 g는 프로토타입 체인에 따라 g 객체 자신이 지니는 멤버, Grade의 prototype에 있는 멤버, Array.prototype에 있는 멤버, 마지막으로 Object.prototype에 있는 멤버까지 접근(3단계)할 수 있습니다.