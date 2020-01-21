# 불변객체와 undefined, null

## 🤘 목표

- [ ]

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

   target["c"] = 3;

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
     name: "soulcactus",
     gender: "female"
   };

   const changeName = (user, newName) => {
     const newUser = user;

     newUser["name"] = newName;

     return newUser;
   };

   const user2 = changeName(user, "hwizzzang"); // 우리는 이미 무슨 일이 일어날 지 알고 있습니다! 😁

   if (user !== user2) {
     console.log("유저 정보가 변경되었습니다.");
   }

   console.log(user["name"], user2["name"]); // hwizzzang hwizzzang
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
     name: "soulcactus",
     gender: "female"
   };

   const changeName = (user, newName) => {
     return {
       name: newName,
       gender: user["gender"]
     };
   };

   const user2 = changeName(user, "hwizzzang");

   if (user !== user2) {
     console.log("유저 정보가 변경되었습니다."); // 이제 작동합니다! 👍
   }

   console.log(user["name"], user2["name"]); // soulcactus hwizzzang
   console.log(user === user2); // false
   ```

   changeName 함수가 새로운 객체를 반환하도록 수정했습니다.
   이제 user와 user2는 서로 다른 객체이므로 안전하게 변경 전과 후를 비교할 수 있습니다.
   다만 아직 미흡한 점이 보이는데요, changeName 함수가 새로운 객체를 만들면서 변경할 필요가 없는 기족 객체의 프로퍼티(이 경우에는 gender)를 하드코딩으로 입력한 점입니다.
   이런 방식으로는 대상 객체에 정보가 많을 수록, 변경해야 할 정보가 많을 수록 수고가 늘어납니다.
   이런 방식보다는 대상 객체의 프로퍼티 수에 상관 없이 모든 프로퍼티를 복사하는 함수를 만드는 편이 더 좋습니다.
   다음 코드를 살펴보겠습니다.

   ```javascript
   const copyObject = target => {
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
     name: "soulcactus",
     gender: "female"
   };

   const user2 = copyObject(user);

   user2["name"] = "hwizzzang";

   if (user !== user2) {
     console.log("유저 정보가 변경되었습니다."); // 이제 작동합니다! 👍
   }

   console.log(user["name"], user2["name"]); // soulcactus hwizzzang
   console.log(user === user2); // false
   ```

   copyObject 함수를 이용해 간단하게 객체를 복사하고 내용을 수정하는 데 성공했습니다.
   이제부터 협업하는 모든 개발자들이 user 객체 내부 변경이 필요할 때 무조건 copyObject 함수를 사용하기로 합의하고 그 규칙을 지킨다는 전제하에서는 user 객체가 곧 불변객체라고 할 수 있겠습니다.
