# 불변객체와 undefined, null

## 불변객체

불변 객체는 최근 React, Vue.js, Angular 등의 라이브러리나 프레임워크에서 뿐만 아니라 함수형 프로그래닝, 디자인 패턴 등에서도 매우 중요한 기초가 되는 개념입니다. 데이터 자체를 변경하고자하면(새로운 데이터를 할당하고자 하면) 기본형 데이터와 마찬가지로 **기존 데이터는 변하지 않습니다.**
그렇다면 내부 프로퍼티를 변경할 필요가 있을 때마다 매번 새로운 객체를 만들어 재할당하기로 규칙을 정하거나, 자동으로 새로운 객체를 만드는 도구를 활용한다면 객체 역시 불변성을 확보할 수 있을 것입니다. 

* 대표적으로 immutable.js, immer.js, immutability-helper등의 라이브러리와, ES6의 spread operator, Object.assign 메서드 등)
혹은 불변성을 확보할 필요가 있을 경우에는 불변 객체로 취급하고, 그렇지 않은 경우에는 기존 방식대로 사용하는 식으로 상황에 따라 대처해도 됩니다.

혹은 불변성을 확보할 필요가 있을 경우에는 불변 객체로 취급하고, 그렇지 않은 경우에는 기존 방식대로 사용하는 식으로 상황에 따라 대처해도 됩니다.

값으로 전달받은 객체에 변경을 가하더라도 원본 객체는 변하지 않아야 하는 경우가 종종 발생하는데요. 바로 이럴 때 불변 객체가 필요합니다. 밑 예제를 같이 보겠습니다.

```javascript
var user = {
    name: "hwi",
    gender: "fame"
};

var changeName = function(user, newName) {
    var newUser = user;
    newUser.name = newName;
    return newUser;
}

var user2  = changeName(user, "zzzang");

if(user !== user2) {
    console.log("유저 정보가 변경되었습니다.");
}

console.log(user.name, user2.name);
console.log(user === user2)
```

위의 예제는 객체의 가변성으로 인한 문제점을 보여주는 간단한 예시입니다. 원래의 의도처럼 정보가 바뀐 시점에 알림을 보내야 한다거나, 바뀌기 전의  정보와 바뀐 후의 정보 차이를 가시적으로 보여줘야 하는 등의 기능을 구현하려면 변경 전과 후에 서로 다른 객체를 바라보게 만들어야합니다. 수정한 코드는 다음과 같습니다.

```javascript
var user = {
    name: "hwi",
    gender: "fame"
};

var changeName = function(user, newName) {
    return {
        name : newName,
        gener : user.gender
    }
}

var user2  = changeName(user, "zzzang");

if(user !== user2) {
    console.log("유저 정보가 변경되었습니다."); // 유저 정보가 변경되었습니다.
}

console.log(user.name, user2.name); // hwi zzzang
console.log(user === user2) // false
```

changeName 함수가 새로운 객체를 반환하도록 수정했습니다. 이제 user와 user2는 서로 다은 객체이므로 안전하게 변경 전과 후를 비교할 수 있습니다. 다만 위에서는 변경할 필요가 없는 기존 객체의 프로퍼티(gender)가 입력되었는데요. 이보다는 대상 객체의 프로퍼티 개수에 상관 없이 모든 프로퍼티를 복사하는 함수를 만드는 편이 더 좋습니다.

```javascript
// 기존 정보를 복사하여 새로운 객체를 반환하는 함수(얕은 복사)
var copyObject = function(target) {
    var result = {};
    for(var prop in target) {
        result[prop] = target[prop];
    }
    return result;
}

var user = {
    name: "hwi",
    gender: "fame"
};

var user2  = copyObject(user);
user2.name = "zzzang";

if(user !== user2) {
    console.log("유저 정보가 변경되었습니다."); // 유저 정보가 변경되었습니다.
}

console.log(user.name, user2.name); // hwi zzzang
console.log(user === user2) // false
```

copyObject는 for in 문법을 이용해 result 객체에 target 객체의 프로퍼티를 복사하는 함수입니다.
copyObject 함수를 통해 간단하게 객체를 복사하고 내용을 수정하는 데 성공했습니다. 위의 코드로 협업하는 모든 개발자들이 user 객체 내부의 변경이 필요할 때는 무조건 copuObject 함수를 사용하기로 합의하고 그 규칙을 지킨다는 전제하에는 user 객체가 곧 불변 객체라고 볼 수 있는데요.

그렇지만 그보다 모두가 그 규칙을 따르지 않고는 프로퍼티 변경할 수 없게끔 시스템적으로 제약을 거는 편이 안전할 것입니다. 이런 맥락에서 자바스크립트 내장 객체가 아닌 라이브러리 자체에서 불변성을 지닌 별도의 데이터 타입과 그에 따른 메서드를 제공하는 immutable.js, baobab.js 등의 라이브러리들이 인기를 끌고있습니다.

위의 예제는 얕은 복사만을 수행한다는 부분이 가장 아쉬운데, 이 부분을 보완하는 방법을 소개하는 것으로 해당 내용을 마치겠습니다.

## 얕은 복사와 깊은 복사

얕은 복사는 바로 아래 단계의 값만 복사하는 방법이고, 깊은 복사는 내부의 모든 값들을 하나하나 찾아 전부 복사하는 방법입니다. 앞의 예제에서는 얕은 복사만 수행했는데요. 이 말은 중첩된 객체에서 참조형 데이터가 저장된 프로퍼티를 복사할 때 그 주솟값만 복사한다는 의미입니다. 그러면 해당 프로퍼티에 대해 원본과 사본이 모두 동일한 참조형 데이터의 주소를 가리키게 되겠죠. 사본을 바꾸면 원본도 바뀌고, 원본을 바뀌면 사본도 바뀝니다.

```javascript
var copyObject = function(target) {
    var result = {};
    for(var prop in target) {
        result[prop] = target[prop];
    }
    return result;
}

var user = {
    name: 'hwizzzang',
    urls: {
        portfolio: 'https://github.com/hwizzzang',
        blog: 'https://hwizzzang.netlify.com',
        twitter: 'https://twitter.com/_hwizzzang',
    },
};

var user2 = copyObject(user);

user2['name'] = 'zzzanghwi';
console.log(user['name'] === user2['name']); // false

user['urls']['portfolio'] = 'https://github.com/zzzanghwi';
console.log(user['urls']['portfolio'] === user2['urls']['portfolio']); // true

user2['urls']['blog'] = '';
console.log(user['urls']['blog'] === user2['urls']['blog']); // true
```

user2의 name 프로퍼티를 바꿔도 user의 name 프로퍼티는 바뀌지 않았습니다. 반면 portfolio, blog를 바꾼 줄에서는 원본과 사본 중 어느 쪽을 바꾸더라도 다른 한쪽의 값도 함께 바뀐것을 확인할 수 있는데요.

즉 user 객체에 직접 속한 프로퍼티에 대해서는 복사해서 완전히 새로운 데이터가 만들어진 반면, 한 단계 더 들어간 urls의 내부 프로퍼티들은 **기존 데이터를 그대로 참조**하는 것입니다.

이런 현상이 발생하지 않게 하려면 user.urls 프로퍼티에 대해서도 불변 객체로 만들 필요가 있습니다.
다음 코드를 살펴보겠습니다.

```javascript
var copyObject = function(target) {
    var result = {};
    for(var prop in target) {
        result[prop] = target[prop];
    }
    return result;
}

var user = {
    name: 'hwizzzang',
    urls: {
        portfolio: 'https://github.com/hwizzzang',
        blog: 'https://hwizzzang.netlify.com',
        twitter: 'https://twitter.com/_hwizzzang',
    },
};

var user2 = copyObject(user);
user2['urls'] = copyObject(user['urls']);

user['urls']['portfolio'] = 'https://github.com/zzzanghwi';

console.log(user['urls']['portfolio'] === user2['urls']['portfolio']); // false

user2['urls']['blog'] = '';

console.log(user['urls']['blog'] === user2['urls']['blog']); // false
```

user의 urls 프로퍼티에 copyObject 함수를 실행한 결과를 할당했습니다.
이제 urls 프로퍼티의 내부까지 복사하여 새로운 데이터가 만들어졌으므로 user2의 urls는 user의 urls를 참조하지 않습니다.

그러니까 어떤 객체를 복사할 때 객체 내부의 모든 값을 복사해서 완전히 새로운 데이터를 만들고자 할 때, 객체의 프로퍼티 중에서 그 값이 **기본형 데이터일 경우에는 그대로 복사하면 되지만 참조형 데이터일 경우에는 다시 그 내부의 프로퍼티들을 복사**해야 합니다. 이 과정을 참조형 데이터가 있을 때마다 재귀적으로 수행해야만 비로소 깊은 복사가 되는 것입니다.

이 개념을 바탕으로 copyObject 함수를 깊은 복사 방식으로 수정해 보겠습니다.

```javascript
var copyObjectDeep = (target) => {
    var result = {};

    if (typeof target === 'object' && target !== null) {
        for (const prop in target) {
            result[prop] = copyObjectDeep(target[prop]);
        }
    } else {
        result = target;
    }

    return result;
};
```

target이 객체인 경우에는 내부 프로퍼티들을 순회하며 copyObjectDeep 함수를 재귀적으로 호출하고, 객체가 아닌 경우에는 target을 그대로 지정하게끔했습니다. 이 함수를 사용해 객체를 복사한 다음에는 원본과 사본이 서로 완전히 다른 객체를 참조하게 되어 어느 쪽의 프로퍼티를 변경하더라도 다른 쪽에 영향을 주지 않습니다.

```javascript
var copyObjectDeep = (target) => {
    var result = {};

    if (typeof target === 'object' && target !== null) {
        for (const prop in target) {
            result[prop] = copyObjectDeep(target[prop]);
        }
    } else {
        result = target;
    }

    return result;
};

var obj = {
    a: 1,
    b: {
        c:null,
        d:[1,2]
    }
};

var obj2 = copyObjectDeep(obj)

obj2.a = 3;
obj2.b.c = 4;
obj.b.d[1] = 3;

console.log(obj); // { a: 1, b: { c: null, d: [1, 2] } }
console.log(obj2); // { a: 3, b: { c: 4, d: {0: 1, 1: 3}} }
```

추가로 hasOwnProperty 메서드를 활용해 프로토타입 체이닝을 통해 상속된 프로퍼티를 복사하지 않게끔 할 수도 있습니다.

```javascript
var obj = {
    a: 1,
    b: {
        c: null,
        d: [1, 2],
    },
};

console.log(obj.hasOwnProperty('b')); // true
console.log(obj.hasOwnProperty('c')); // false
```

ES5의 getter와 setter를 복사하는 방법은 안타깝게도 ES6의 Object.getOwnPropertyDescriptor또는 ES2017의 Object.getOwnPropertyDescriptor외에는 마땅한 방법이 없습니다.

끝으로 간단하게 깊은 복사를 처리할 수 있는 다른 방법 하나를 더 소개하겠습니다.객체를 JSON 문법으로 표현된 문자열로 변환했다가 다시 JSON 객체로 바꾸는 방법입니다.
다만, 메서드(함수)나 숨겨진 프로퍼티인 \_\_proto\_\_나 getter, setter 등과 같이 JSON으로 변경할 수 없는 프로퍼티들은 모두 무시합니다. httpRequest로 받은 데이터를 저장한 객체를 복사할 때 등 순수한 정보만 다룰 때 활용하기 좋은 방법입니다.

```javascript
var copyObjectViaJSON = (target) => {
    return JSON.parse(JSON.stringify(target));
};

var obj = {
    a: 1,
    b: {
        c: null,
        d: [1, 2],
        func1: function() {
            console.log(3);
        },
        func2: function() {
            console.log(4);
        },
    },
};

var obj2 = copyObjectViaJSON(obj);

obj2['a'] = 3;
obj2['b']['c'] = 4;
obj['b']['d'][1] = 3;

console.log(obj); // { a: 1, b: { c: null, d: [1, 3], func1: f(), func2: f() }}
console.log(obj2); // { a: 3, b: { c: 4, d: [1, 2] } }
```

## undefined와 null

자바스크립트에는 *없음*을 나타내는 undefined와 null이 있습니다.

undefined는 사용자가 명시적으로 지정할 수도 있지만 값이 존재하지 않을 때 자바스크립트 엔진이 자동으로 부여하는 경우도 있습니다.

자바스크립트 엔진은 사용자가 어떤 값을 지정할 것이라고 예상되는 상황임에도 실제로는 그렇게 하지 않았을 때 undefined를 반환합니다. 밑의 세 경우가 이에 해당합니다.

1. 값을 대입하지 않은 변수, 즉 데이터 영역의 메모리 주소를 지정하지 않은 식별자에 접근할 때

2. 객체 내부의 존재하지 않는 프로퍼티에 접근하려고 할 때

3. return 문이 없거나 호출되지 않는 함수 실행의 결과

```javascript
// 자동으로 undefined를 부여하는 경우
var a;
console.log(a) // (1) undefined. 값을 대입하지 않은 변수에 접근

var obj = { a: 1 };
console.log(obj['a']); // 1
console.log(obj['b']); // (2) 존재하지 않는 프로퍼티에 접근=
console.log(b) // c. ReferenceError : b is not defined

var func = function() {};
var c = func(); // (3) return 값이 없으면 undefined를 반환한 것으로 간주.
console.log(c); // undefined
```

값을 대입하지 않은 경우에 대해 배열의 경우에는 조금 특이한 동작을 합니다. 밑의 예제를 보겠습니다.

```javascript
var arr1 = [];
arr1.length = 3;
console.log(arr1); // [empty x 3];

var arr2 = new Array(3);
console.log(arr2); // [empty x 3];

var arr3 = [undefined, undefined, undefined];
console.log(arr3); // [undefined, undefined, undefined]
```

1번쨰 줄에서 빈 배열을 만들고, 2번쨰 줄에서 배열의 크기를 3으로 했더니 3번째 줄에서 [empty x 3]이 출력됐습니다. 이는 배열에 3개의 빈 요소를 확보했지만 확보된 각 요소에는 문자 그대로 어떤 값도, 심지어 indefined 조차도 할당돼 있지 않음을 의미합니다.

반면 8번째 줄에서는 리터럴 방식으로 배열을 생성하여 각 요소에 undefined를 부여하면 세 개의 세 개의 undefined가 담긴 배열을 반환합니다.

이처럼 *비어있는 요소*와 *undefined*를 할당한 요소는 출력 결과가 다르며, 빈 요소는 배열 메서드의 순회 대상에서 제외됩니다. 밑의 코드를 통해 확인해보겠습니다.