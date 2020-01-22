# 불변객체와 undefined, null

## 🤘 목표

-   [ ] 불변객체에 대해 이해한다.
-   [ ] 얕은 복사와 깊은 복사가 무엇이고 어떤 차이점이 있는지 이해한다.
-   [ ] undefined와 null이 무엇이고 어떤 차이점이 있는지 이해한다.

####

## 🔥 불변객체

1. 불변객체란?

    불변객체는 최근 React, Vue, Angular 등의 라이브러리나 프레임워크에서뿐만 아니라 함수형 프로그래밍, 디자인 패턴 등에서도 매우 중요한 기초가 되는 개념입니다.
    앞서 말씀드린 참조형 데이터의 *가변*은 데이터 자체가 아닌 데이터 내부 프로퍼티를 변경할 때만 성립합니다.
    데이터 자체를 변경하고자 하면(새로운 데이터를 할당하고자 하면) 기본형 데이터와 마찬가지로 **기존 데이터는 변하지 않습니다.**
    그렇다면 내부 프로퍼티를 변경할 필요가 있을 때마다 매번 새로운 객체를 만들어 재할당하기로 규칙을 정하거나,
    자동으로 새로운 객체를 만드는 도구(immutable.js, immer,js, immutability-helper 등의 라이브러리나 ES6의 [spread operator](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Spread_syntax), [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 등)를 활용한다면 불변성을 확보할 수 있습니다.
    혹은 불변성을 확보할 필요가 있는 경우에는 불변객체로 취급하고, 그렇지 않은 경우에는 기존 방식대로 사용하는 식으로 상황에 따라 대처할 수 있습니다.
    불변객체가 필요한 상황에 대해 알아보기 전에 먼저, 앞서 언급한 spread operator와 Object.assign 메서드를 살펴보기로 하겠습니다.

    ```javascript
    const arr1 = [1, 2, 3];
    ```

    먼저 arr1을 위와 같이 선언합니다.

    ```javascript
    const arr1 = [1, 2, 3];
    const arr2 = arr1;

    arr2[0] = 4;

    console.log(arr1); // [4, 2, 3];
    console.log(arr2); // [4, 2, 3];
    ```

    이전 파트에서 학습했듯이 arr2에 arr1을 위와 같이 할당하면 *복사*가 아닌 *참조*가 일어납니다.
    하지만 전개 구문을 이용해 아래와 같이 할당하면 어떻게 될까요?

    ```javascript
    const arr1 = [1, 2, 3];
    const arr2 = [...arr1];

    arr2[0] = 4;

    console.log(arr1); // [1, 2, 3];
    console.log(arr2); // [4, 2, 3];
    ```

    *참조*가 아니라 *복사*되었습니다.
    이렇듯 값으로 전달받은 객체에 변경을 가하더라도 원본 객체는 변하지 않아야 하는 경우에 불변객체가 필요합니다.
    Object.assign도 살펴보겠습니다.

    ```javascript
    const target = {};
    const source = { a: 1, b: 2 };

    Object.assign(target, source); // taget에 source를 할당!

    console.log(target); // { a: 1, b: 2 }
    console.log(source); // { a: 1, b: 2 }

    target['c'] = 3;

    console.log(target); // { a: 1, b: 2, c: 3 }
    console.log(source); // { a: 1, b: 2 }
    ```

    역시 *참조*가 아니라 *복사*되었습니다.
    Object.assign은 다음과 같은 방법으로도 사용 가능합니다.

    ```javascript
    const target = { a: 1, b: 2 };
    const source = { b: 3, c: 4 };

    Object.assign(target, source); // taget에 source를 할당!

    console.log(target); // { a: 1, b: 3, c: 4 }
    console.log(source); // { b: 3, c: 4 }
    ```

2. 불변객체가 필요한 상황

    코드를 먼저 살펴본 다음 설명드리도록 하겠습니다.

    ```javascript
    const user = {
        name: 'soulcactus',
        gender: 'female',
    };

    const changeName = (user, newName) => {
        const newUser = user;

        newUser['name'] = newName;

        return newUser;
    };

    const user2 = changeName(user, 'hwizzzang'); // 우리는 이미 무슨 일이 일어날 지 알고 있습니다! 😁

    if (user !== user2) {
        console.log('유저 정보가 변경되었습니다.');
    }

    console.log(user['name'], user2['name']); // hwizzzang hwizzzang
    console.log(user === user2); // true
    ```

    newUser에 user를 할당하는 순간 우리는 *참조*가 일어난다는 사실을 이제 알고 있습니다.
    ~~기억이 나지 않으신다면~~ [part01](././01_week/part01/README.md)을 참고해 주세요.
    조건문을 살펴보시면 "유저 정보가 변경되면 알려주자!"는 이 로직의 목표가 명확하게 보입니다.
    ~~안타깝지만 현재 코드로는 가능할 것 같지 않습니다(...)~~
    위와 같이 정보가 바뀐 시점에 알림을 보내야 한다거나, 바뀌기 전의 정보와 바뀌기 전의 정보 차이를 가시적으로 보여줘야 하는 등의 기능을 구현해야 한다면 이대로는 안되므로 변경 전과 후에 서로 다른 객체를 바라보게 만들어야 합니다.
    다음 코드를 살펴보겠습니다.

    ```javascript
    const user = {
        name: 'soulcactus',
        gender: 'female',
    };

    const changeName = (user, newName) => {
        return {
            name: newName,
            gender: user['gender'],
        };
    };

    const user2 = changeName(user, 'hwizzzang');

    if (user !== user2) {
        console.log('유저 정보가 변경되었습니다.'); // 이제 작동합니다! 👍
    }

    console.log(user['name'], user2['name']); // soulcactus hwizzzang
    console.log(user === user2); // false
    ```

    changeName 함수가 새로운 객체를 반환하도록 수정했습니다.
    이제 user와 user2는 서로 다른 객체이므로 안전하게 변경 전과 후를 비교할 수 있습니다.
    다만 아직 미흡한 점이 보이는데요, changeName 함수가 새로운 객체를 만들면서 변경할 필요가 없는 기족 객체의 프로퍼티(이 경우에는 gender)를 하드코딩으로 입력한 점입니다.
    이런 방식으로는 대상 객체에 정보가 많을 수록, 변경해야 할 정보가 많을 수록 수고가 늘어납니다.
    이런 방식보다는 대상 객체의 프로퍼티 수에 상관 없이 모든 프로퍼티를 복사하는 함수를 만드는 편이 더 좋습니다.
    다음 코드를 살펴보겠습니다.

    ```javascript
    const copyObject = (target) => {
        const result = {};

        for (const prop in target) {
            result[prop] = target[prop];
        }

        return result;
    };
    ```

    copyObject 함수는 for in 문법을 이용해 result 객체에 target 객체 프로퍼티들을 복사하는 함수입니다.
    프로토타입 체이닝 상의 모든 프로퍼티를 복사하는 점, getter, setter는 복사하지 않는 점, 얕은 복사(뒷 부분에서 살펴보겠습니다)만 수행한다는 점에서 약간의 아쉬움이 있으나
    그 아쉬움을 보완하기 위해 함수를 무겁게 만드는 것보다 일단 이 예제의 객체에 대해서는 문제되지 않으므로 진행해보도록 합니다.

    ```javascript
    const user = {
        name: 'soulcactus',
        gender: 'female',
    };

    const user2 = copyObject(user);

    user2['name'] = 'hwizzzang';

    if (user !== user2) {
        console.log('유저 정보가 변경되었습니다.'); // 이제 작동합니다! 👍
    }

    console.log(user['name'], user2['name']); // soulcactus hwizzzang
    console.log(user === user2); // false
    ```

    copyObject 함수를 이용해 간단하게 객체를 복사하고 내용을 수정하는 데 성공했습니다.
    이제부터 협업하는 모든 개발자들이 user 객체 내부 변경이 필요할 때 무조건 copyObject 함수를 사용하기로 합의하고 그 규칙을 지킨다는 전제하에서는 user 객체가 곧 불변객체라고 할 수 있겠습니다.
    그렇지만 모두가 그 규칙을 지키리라는 신뢰에만 의지하는 것은 얇고 깨지기 쉬운 살얼음판을 걷는 것과 같습니다.
    그보다는 모두가 그 규칙을 따르지 않고는 프로퍼티를 변경할 수 없게끔 시스템적으로 제약을 거는 편이 안전합니다.
    이 같은 맥락에서 자바스크립트 내장 객체가 아닌 라이브러리 자체에 불변성을 지닌 별도의 데이터 타입과 그에 따른 메서드를 제공하는 immutable.js, baobab.js 등의 라이브러리가 등장해 인기를 끌고 있습니다.

####

## 📄 얕은 복사와 깊은 복사

얕은 복사란 바로 아래 단계의 값만 복사하는 방법이고, 깊은 복사란 내부의 모든 값들을 하나하나 찾아서 전부 복사하는 방법입니다.
앞서 copyObject 함수는 얕은 복사만 수행했습니다.
이 말은 중첩된 객체에서 참조형 데이터가 저장된 프로퍼티를 복사할 때 그 주솟값만 복사한다는 의미입니다.
그러면 해당 프로퍼티에 대해 원본과 사본이 모두 동일한 참조형 데이터의 주소를 가리키게 됩니다.
사본을 바꾸면 원본도 바뀌고, 원본을 바꾸면 사본도 바뀝니다.

```javascript
const user = {
    name: 'soulcactus',
    urls: {
        portfolio: 'https://github.com/soulcactus',
        blog: 'https://soulcactus.netlify.com',
        twitter: 'https://twitter.com/_soulcactus',
    },
};

const user2 = copyObject(user);

user2['name'] = 'hwizzzang';

console.log(user['name'] === user2['name']); // false

user['urls']['portfolio'] = 'https://github.com/hwizzzang';

console.log(user['urls']['portfolio'] === user2['urls']['portfolio']); // true

user2['urls']['blog'] = '';

console.log(user['urls']['blog'] === user2['urls']['blog']); // true
```

얕은 복사가 일어났기 때문에 user2의 urls가 user의 urls를 그대로 참조하고 있습니다.
이런 현상이 발생하지 않게 하려면 user.urls 프로퍼티에 대해서도 불변 객체로 만들 필요가 있습니다.
다음 코드를 살펴보겠습니다.

```javascript
const user2 = copyObject(user);

user2['urls'] = copyObject(user['urls']);
user['urls']['portfolio'] = 'https://github.com/hwizzzang';

console.log(user['urls']['portfolio'] === user2['urls']['portfolio']); // false

user2['urls']['blog'] = '';

console.log(user['urls']['blog'] === user2['urls']['blog']); // false
```

user의 urls 프로퍼티도 copyObject를 통해 복사해 user2의 urls에 할당했습니다.
이제 user2의 urls는 user의 urls를 참조하지 않습니다.
그러니까 어떤 객체를 복사할 때 객체 내부의 모든 값을 복사해서 완전히 새로운 데이터를 만들고자 할 때,
객체의 프로퍼티 중에서 그 값이 기본형 데이터일 경우에는 그대로 복사하면 되지만 참조형 데이터일 경우에는 다시 그 내부의 프로퍼티들을 복사해야 합니다.
이 과정을 참조형 데이터가 있을 때마다 재귀적으로 수행해야만 비로소 깊은 복사가 되는 것입니다.
이 개념을 바탕으로 copyObject 함수를 깊은 복사 방식으로 수정해 보겠습니다.

```javascript
const copyObjectDeep = (target) => {
    let result = {};

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

target !== null 조건을 붙인 이유는 typeof 명령어가 null에 대해서도 object를 반환하기 때문입니다.
이는 자바스크립트 자체 버그입니다.
아래 코드를 통해 확인해 보도록 하겠습니다.

```javascript
const obj = {
    a: 1,
    b: {
        c: null,
        d: [1, 2],
    },
};

const obj2 = copyObjectDeep(obj);

obj2['a'] = 3;
obj2['b']['c'] = 4;
obj2['b']['d'][1] = 3;

console.log(obj); // { a: 1, b: { c: null, d: [1, 2] } }
console.log(obj2); // { a: 3, b: { c: 4, d: {0: 1, 1: 3}} }
```

추가로 [hasOwnProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) 메서드를 활용해 프로토타입 체이닝을 통해 상속된 프로퍼티를 복사하지 않게끔 할 수도 있습니다.

```javascript
const obj = {
    a: 1,
    b: {
        c: null,
        d: [1, 2],
    },
};

console.log(obj.hasOwnProperty('b')); // true
console.log(obj.hasOwnProperty('c')); // false
```

ES5의 getter와 setter를 복사하는 방법은 안타깝게도 ES6의 [Object.getOwnPropertyDescriptor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor) 또는 ES2017의 [Object.getOwnPropertyDescriptor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors) 외에는 마땅한 방법이 없습니다.

끝으로 간단하게 깊은 복사를 처리할 수 있는 다른 방법 하나를 더 소개하겠습니다.
원리도 단순한데요, 객체를 JSON 문법으로 표현된 문자열로 변환했다가 다시 JSON 객체로 바꾸는 방법입니다.
이 방법은 단순함에도 불구하고 잘 동작합니다.
다만, 메서드(함수)나 숨겨진 프로퍼티인 **proto**나 getter, setter 등과 같이 JSON으로 변경할 수 없는 프로퍼티들은 모두 무시합니다.
httpRequest로 받은 데이터를 저장한 객체를 복사할 때 등 순수한 정보만 다룰 때 활용하기 좋은 방법입니다.

```javascript
const copyObjectViaJSON = (target) => {
    return JSON.parse(JSON.stringify(target));
};

const obj = {
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

const obj2 = copyObjectViaJSON(obj);

obj2['a'] = 3;
obj2['b']['c'] = 4;
obj['b']['d'][1] = 3;

console.log(obj); // { a: 1, b: { c: null, d: [1, 3], func1: f(), func2: f() }}
console.log(obj2); // { a: 3, b: { c: 4, d: [1, 2] } }
```

####

## ❎ undefined와 null

자바스크립트에서는 *없음*을 나타내는 값이 두 가지가 있는데요, 바로 undefined와 null입니다.
두 값은 *없다*는 의미는 같은 듯하나 미세하게 다르고 사용하는 목적 또한 다릅니다.

1. undefined

    undefined는 사용자가 명시적으로 지정할 수 있으나 값이 존재하지 않을 때 자바스크립트 엔진이 자동으로 부여하는 경우도 있습니다.
    자바스크립트 엔진은 사용자가 응당 어떤 값을 지정할 것이라고 예상되는 상황임에도 실제로 그렇게 하지 않았을 때 undefined를 반환합니다.
    다음 세 가지 경우가 이에 해당합니다.

    - 값을 대입하지 않은 변수, 즉 데이터 영역의 메모리 주소를 지정하지 않은 식별자에 접근할 때
    - 객체 내부의 존재하지 않는 프로퍼티에 접근하려고 할 때
    - return문이 없거나 호출되지 않는 함수의 실행 결과

    ```javascript
    let a;

    console.log(a); // undefined

    const obj = { a: 1 };

    console.log(obj['a']); // 1
    console.log(obj['b']); // undefined
    console.log(b); // Uncaught ReferenceError: b is not defined

    const func = () => {}; // return 값이 없으면 undefined를 반환한 것으로 간주
    const c = func();

    console.log(c); // undefined
    ```

    undefined를 반환하는 경우와 레퍼런스 에러가 나는 경우의 차이를 이해하려면, not defined는 *정의된 적이 없음*을 의미하고 undefined는 *아직 정의되지 않았음*으로 이해하는 것이 쉽습니다.
    변수 a는 이미 선언됐으나 _아직_ 값을 가지지 않은 경우이므로 undefined를 반환합니다.
    obj['b'] 역시 _아직 정의되지 않았으므로_ undefined를 반환합니다.
    변수 b는 _정의된 적이 없으므로_ 레퍼런스 에러가 일어납니다.
    참고로 변수 a를 선언하고 아무런 값도 할당하지 않았을 때, 다른 도서에서는 "let a라는 구문에 의해 식별자 a에 자동으로 undefined가 할당되고, 그 값이 할당되지 않은 식별자에 접근할 때"
    자동으로 할당된 undefined를 반환한다고 하지만, 자바스크립트는 실제로 그렇게 동작하지 않습니다.
    정확히는 아무것도 할당하지 않고 끝나며, 이후 변수 a에 접근하고자 할 때 비로소 undefined를 반환하는 것이 맞습니다.

    값을 대입하지 않은 배열의 경우 조금 특이한 동작을 하는데요, 코드를 통해 살펴보겠습니다.

    ```javascript
    const arr1 = [];

    console.log(arr1); // []

    arr1.length = 3;
    console.log(arr1); // [empty x 3];

    const arr2 = new Array(3);

    console.log(arr2); // [empty x 3];

    const arr3 = [undefined, undefined, undefined];

    console.log(arr3); // [undefined, undefined, undefined]
    ```

    빈 배열을 선언한 후 길이를 지정하면 undefined 조차 들어 있지 않은 세 개의 빈 요소가 담긴 배열을 반환합니다.
    반면, 세 개의 undefined가 담긴 배열을 할당하면 세 개의 undefined가 담긴 배열이 반환됩니다.
    이처럼 비어 있는 요소와 undefined를 할당한 요소는 출력 결과가 다르며, 빈 요소는 배열 메서드의 순회 대상에서 제외됩니다.
    다음 코드를 통해 살펴보겠습니다.

    ```javascript
    const arr1 = [undefined, 1];
    const arr2 = [];

    arr2[1] = 1;

    arr1.forEach((v, i) => console.log(v, i)); // undefined 0 / 1 1
    arr2.forEach((v, i) => console.log(v, i)); // 1 1

    arr1.map((v, i) => v + i); // [NaN, 2]
    arr2.map((v, i) => v + i); // [empty, 2]

    arr1.filter((v) => !v); // [undefined]
    arr2.filter((v) => !v); // []

    arr1.reduce((p, c, i) => {
        return p + c + i;
    }, ''); // undefined011

    arr2.reduce((p, c, i) => {
        return p + c + i;
    }, ''); // 11
    ```

    직접 undefined를 할당한 arr1에 대해서는 일반적으로 알고 있는 대로 배열의 모든 요소를 순회해 결과를 출력합니다.
    그러나 arr2에 대한 결과를 보면, 각 메서드들이 비어 있는 요소에 대해서는 어떠한 처리도 하지 않고 건너뛰었음을 알 수 있습니다.
    이러한 동작이 배열에서만 발견할 수 있는 특별한 현상인 것처럼 소개했지만, 사실 배열도 *객체*임을 생각해 보면 지극히 자연스러운 현상입니다.
    존재하지 않는 프로퍼티에 대해서는 순회할 수 없는 것이 당연합니다.
    배열은 무조건 length 프로퍼티의 개수만큼 빈 공간을 확보하고 각 공간에 인덱스를 이름으로 지정할 것이라고 생각하기 쉽지만,
    실제로는 객체와 마찬가지로 특정 인덱스에 값을 지정하 때 비로소 빈 공간을 확보하고 인덱스를 이름으로 지정하고 데이터의 주소값을 저장하는 등의 동작을 합니다.
    즉, 값이 지정되지 않은 인덱스는 _아직_ 존재하지 않는 프로퍼티에 지나지 않습니다.
    그렇다면 사용자가 명시적으로 부여한 경우와 비어있는 요소에 접근하려 할 때 반환되는 두 경우의 'undefined'의 의미를 구분할 수 있겠습니다.
    사용자가 명시적으로 지정한 경우, undefined는 그 자체로 값입니다.
    비록 비어있음을 의미하긴 하지만 하나의 값으로 동작하기 때문에 이때의 프로퍼티나 배열의 요소는 고유의 키값(프로퍼티 이름)이 실존하게 되고 따라서 순회의 대상입니다.
    한편 자바스크립트 엔진이 하는 수 없이 반환하는 undefined의 경우 해당 프로퍼티 내지 배열의 키값 자체가 존재하지 않음을 의미합니다.
    종합해 말하면 값으로써 할당된 undefined는 실존 데이터인 반면, 자바스크립트 엔진이 반환하는 undefined는 말 그래도 값이 없음을 나타냅니다.

2. null

    사실, 같은 의미를 가진 null이 별도로 있기 때문에 굳이 undefined를 써야할 이유가 없습니다.
    비어있음을 명시적으로 나타내고 싶을 때는 null을 쓰면 됩니다.
    null은 애초부터 이러한 용도로 만들어진 데이터 타입이며, 이 규칙을 따르면 undefined는 오직 "값을 대입하지 않은 변수에 접근하고자 할 때 자바스크립트 엔진이 반환해 주는 값"으로서만 존재할 수 있습니다.
    하지만 null은 한가지 주의할 점이 있습니다.
    앞서 말씀드렸듯, typeof null이 object라는 점입니다.
    따라서 어떤 변수의 값이 null인지 여부를 판별하기 위해서는 다른 방식으로 접근해야 합니다.
    아래 코드를 살펴보겠습니다.

    ```javascript
    const n = null;

    console.log(typeof n); // object

    console.log(n == undefined); // true
    console.log(n == null); // true

    console.log(n === undefined); // false
    console.log(n === null); // true
    ```

    타입을 비교하지 않는 동등 연산자로 비교할 경우 null과 undefined가 서로 같다고 판단하지만 타입을 비교하는 일치 연산자로 비교할 경우 다르다고 판단합니다.

####

## 💬 마치며

데이터 타입 파트가 끝났습니다. 변수와 식별자, 변수를 선언하면 컴퓨터의 메모리 영역에서 일어나는 일, 기본형과 참조형이 무엇이고 어떤 차이점이 있는지, 가변값과 불변값이 무엇이고 어떤 차이점이 있는지, 불변 객체를 생성하는 방법과 얕은 복사, 깊은 복사, undefined와 null까지 알아봤습니다. 자바스크립트의 가장 기초적인 영역인 만큼 확실하게 짚고 넘어가면, 중급 개발자로 성장하기 위한 초석이 될 것입니다. 😃
