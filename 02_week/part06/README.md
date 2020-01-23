# 콜백 함수

## 🧐 콜백 함수란?

콜백 함수는 다른 코드(함수 또는 메서드)에게 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수이며, 콜백 함수를 위임받은 코드는 자체적인 내부 로직에 의해 이 콜백 함수를 적절한 시점에 실행하게됩니다.

* 골백 함수는 다른 코드의 인자로 넘겨주는 함수임

* 콜백 함수를 넘겨 받은 코드는 이 콜백 함수를 필요에 따라 적절한 시점에 실행함

* 콜백 함수는 "제어권"과 관련이 깊음


## 🧐 콜백 함수 제어권

### **콜백 함수 호출 시점**

콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수 호출 시점에 대한 제어권을 가집니다.

```javascript
var count = 0;
var cbFunc = function() {
    console.log(count);
    if(++count > 4) clearInterval(timer);
};

var timer = setInterval(cbFunc, 300)
```

1. count 변수를 선언하고 0을 할당합니다.

2. timer 변수에는 setInterval의 ID 값이 담깁니다.

3. 콜백 함수인 cbFunc 함수 내부에서는 count 값을 출력하고, count를 1만큼 증가시킨 다음, 그 값이 4보다 크면 반복 실행을 종료합니다.

4. setInterval이라고 하는 '다른 코드'에 첫 번째 인자로서 cbFunc 함수를 넘겨주면, 제어권을 넘겨받은 setInterval이 스스로의 판단에 따라 적절한 시점에 이 익명 함수를 실행합니다.

### **콜백 함수 인자**

콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지에 대한 제어권을 가집니다.

예제를 보기 전 map 메서드의 구조를 먼저 살펴보겠습니다.

```javascript
// Map 메서드 구조
Array.prototype.map(callback[, thisArg])
callback: function(currentValue, index, array)
```

* map 메서드는 첫 번째 인자로 callback 함수를 받고, 생략 가능한 두 번째 인자로 콜백 함수 내부에서 this로 인식할 대상을 특정할 수 있습니다.

* thisArg를 생략할 경우 일반적인 함수와 마찬가지로 전역객체가 바인딩됩니다.

* map 메서드는 메서드의 대상이 되는 배열의 모든 요소들을 처음부터 끝까지 하나씩 돌며 콜백 함수를 반복 호출하고, 콜백 함수의 실행 결과들을 모아 새로운 배열을 만듭니다.

* 첫 번째 인자에는 배열의 요소 중 현재값, 두 번째 인자에는 현재값의 인덱스, 세 번째 인자에는 map 메서드의 대상이 되는 배열 자체가 담깁니다.

```javascript
var newArr = [10, 20, 30].map(function (currentValue, index) {
   console.log(currentValue, index);
   return currentValue + 5;
});

console.log(newArr);

// -- 결과 -- 
// 10 0
// 20 1
// 30 2
// [15, 25, 35]
```

1. newArr 변수를 선언하고 우항의 결과를 할당하였습니다. 우항은 배열 [10, 20, 30]에 map 메서드를 호출하고 있습니다. 이때 첫 번째 매개변수로 익명 함수를 전달합니다.

2. 배열 [10, 20, 30]의 각 요소를 처음부터 하나씩 꺼내어 콜백 함수를 실행합니다.

3. 세 번째 요소에 대한 콜백 함수까지 실행을 마치고 나면, [15, 25, 35] 라는 새로운 배열이 만들어져 변수 newArr에 담기고, 이 값이 6번째 줄에서 출력됩니다.

인자의 순서를 임의로 바꾸어 사용한 경우에는 어떤 결과가 나올까요?

```javascript
// currentValue와 index의 순서를 바꿈
var newArr2 = [10, 20, 30].map(function (index, currentValue) {
   console.log(index, currentValue);
   return currentValue + 5;
});

console.log(newArr2);

// -- 결과 -- 
// 10 0
// 20 1
// 30 2
// [5, 6, 7]
```
첫 번째 인자의 이름을 어떤 것으로 하든 관계 없이 그냥 순회 중인 배열 중 현재 요소의 값을 배정합니다.

따라서 newArr2를 출력하였을 때 [15, 25, 35]가 아닌 [5, 6, 7]이라는 결과가 나옵니다.
currentValue라고 명명한 인자의 위치가 두 번째라서 여기에 인덱스 값을 부여했기 때문입니다.

map 메서드를 호출해서 원하는 배열을 얻으려면 map 메서드에 정의된 규칙에 따라 함수를 작성해야 합니다. map 메서드에 정의된 규칙에는 콜백 함수의 인자로 넘어올 값들 및 그 순서도 포함돼 있습니다. 

콜백 함수를 호출하는 주체는 사용자가 아닌 map 메서드이므로 map 메서드가 콜백 함수를 호출할 때 인자에 어떤 값들을 어떤 순서로 넘길 것인지가 전적으로 map 메서드에게 달린 것입니다.

### **콜백 함수 this**

* 콜백 함수도 함수이기 때문에 기본적으로 this가 전역 객체를 참조합니다.
 
* 제어권을 넘겨받을 코드에서 콜백 함수에 별도로 this가 될 대상을 지정한 경우에는 그 대상을 참조합니다.

map 메서드로 구현한 밑의 예제를 통해 별도의 this를 지정하는 방식 및 제어권에 대해 알아보겠습니다.
예외 처리에 대한 내용은 모두 배제하고 동작 원리를 이해하는 것을 목표로 핵심 내용만 작성한 예제입니다.

```javascript
Array.prototype.map = function(callback, thisArg) {
   var mappedArr = [];
   for (var i = 0; i < this.length; i++) {
      var mappedValue = callback.call(thisArg || window, this[i], i, this);
      mappedArr[i] = mappedValue;
   }

   return mappedArr;
}
```

* this에는 thisArg 값이 있을 경우에는 그 값을, 없을 경우에는 전역객체를 지정,
* 첫 번째 인자에는 메서드의 this가 배열을 가리킬 것이므로 배열의 i번째 요소 값을 지정,
* 두 번째 인자에는 i 값 지정,
* 세 번째 인자에는 배열 자체를 지정해 호출합니다. 

제어권을 넘겨받을 코드에서 call/apply 메서드의 첫 번째 인자에 콜백 함수 내부에서의 this가 될 대상을 명시적으로 바인딩하기 때문에 this에 다른 값이 담기는 것을 알 수 있습니다.

<!-- 예제에서는 익명함수의 call() 메소드를 호출하는데, 첫 번째 인자는 greetings 객체 배열의 각 요소를 주고, 두 번째 인자는 함수의 인자로 보낼 인덱스를 주게 됩니다. -->

```javascript
setTimeout(function() { console.log(this); }, 300);  // (1) window {...}

[1, 2, 3, 4, 5].forEach(function(x) {
   console.log(this);     // (2) window {...}
});

document.body.innerHtml += '<button id="a">클릭</button>';
document.body.querySelector("#a")
   .addEventListener("click", function(e) {
      console.log(this, e); // (3) <button id="a">클릭</button> MouseEvent
   })
```

1. (1)의 setTimeout은 내부에서 콜백 함수를 호출할 때 콜백 함수 내부에서의 this가 전역객체를 가리킵니다.

2. (2)의 forEach는 별도의 인자로 this를 넘겨주지 않았기 때문에 전역객체를 가리킵니다. 

3. (3)의 addEventListener는 내부에서 콜백함수를 호출할 때 call 메서드의 첫 번째 인자에 addEventListener 메서드의 this를 그대로 넘기도록 정의돼 있기 때문에 콜백 함수 내부에서의 this가 addEventListener를 호출한 주체인 HTML 엘리먼트를 가리킵니다.

## **콜백 함수는 함수!**

콜백 함수로 어떤 객체의 메서드를 전달하더라도 그 메서드는 메서드가 아닌 함수로서 호출됩니다.

```javascript
var obj = {
   vals : [1, 2, 3],
   logValues: function(v, i) {
      console.log(this, v, i);
   }
};
obj.logValues(1, 2);   // (1) {vals: Array(3), logValues: ƒ} 1 2
[4, 5, 6].forEach(obj.logValues);  // (2) Window { ... } 4 0, 
                                   //     Window { ... } 5 1, 
                                   //     Window { ... } 6 2
```

1. (1)에서 obj 객체의 logValues는 메서드로 정의됐습니다. 7번째 줄에서 메서드의 이름 앞에 점이 있으니 메서드로서 호출한 것입니다. 따라서 this는 obj를 가리키고, 인자로 넘어온 1, 2가 출력됩니다. 

2. logValues 메서드를 forEach 함수의 콜백 함수로서 전달했습니다. obj를 this로 하는 메서드를 그대로 전달한 것이 아니라 obj.logValues가 가리키는 함수만 전달한 것입니다. forEach에 의해 콜백이 함수로서 호출되고, 별도로 this를 지정하는 인자를 지정하지 않았으므로 함수 내부에서의 this는 전역객체를 바라보게 됩니다.

어떤 함수의 인자에 객체의 메서드를 전달하더라도 이는 결국 메서드가 아닌 함수일 뿐입니다. 

객체의 메서드를 콜백 함수로 전달하면 해당 객체를 this로 바라볼 수 없게 된다는 점은 이제 아실 겁니다. 

## **콜백 함수 내부의 this에 다른 값 바인딩하기**

그럼에도 콜백 함수 내부에서 this가 객체를 바라보게 하고 싶다면 어떻게 해야할까요?

별도의 인자로 this를 받는 함수의 경우에는 여기에 원하는 값을 넘겨주면 되지만, 그렇지 않은 경우에는 this의 제어권도 넘겨주게 되므로 사용자가 임의로 값을 바꿀 수 없습니다.

### **bind 메서드 활용하여 콜백 함수 this에 다른 값 바인딩**

```javascript
var obj1 = {
   name : "obj1",
   func : function() {
      console.log(this.name);
   }
};

setTimeout(obj1.func.bind(obj1), 1000);

var obj2 = { name : "obj2" };
setTimeout(obj1.func.bind(obj2), 1500);
```

* bind 메서드는 call과 비슷하지만 즉시 호출하지는 않고 넘겨 받은 this 및 인수들을 바탕으로 새로운 함수를 반환하기만 하는 메서드입니다. 
다시 새로운 함수를 호출할 때 인수를 넘기면 그 인수들은 기존 bind 메서드를 호출할 때 전달했던 인수들의 뒤에 이어서 등록됩니다. 
즉, bind 메서드는 함수에 this를 미리 적용하는 것과 부분 적용 함수를 구현하는 두 가지 목적을 모두 지닙니다.

## **콜백 지옥**

콜백 지옥은 콜백 함수를 익명 함수로 전달하는 과정이 반복되어 코드의 들여쓰기가 감장하기 힘들 정도로 깊어지는 현상으로, 자바스크립트에서 흔히 발생하는 문제입니다.

이 콜백 지옥을 해결할 수 있는 가장 간단한 해결책에는 익명의 콜백 함수를 모두 기명함수로 전환하는 방법이 있습니다.

또 다른 해결 방법에는 Promise, Generator, async/await 등이 있습니다. 이들을 이용한 예시는 밑에서 비동기 제어 관련 글과 함께 확인해보겠습니다.

## **콜백 비동기 제어**

### **동기적 코드**

동기적 코드는 현재 실행 중인 코드가 완료된 후에야 다음 코드를 실행하는 방식입니다. CPU의 계산에 의해 즉시 처리가 가능한 대부분의 코드는 동기적인 코드이며, 계산식이 복잡해서 CPU가 계산하는 데 시간이 많이 필요한 경우라 하더라도 이와 같습니다.

### **비동기적 코드**

비동기적 코드는 현재 실행 중인 코드의 완료 여부와 무관하게 즉시 다음 코드로 넘어갑니다.

별도의 요청, 실행 대기, 보류 등과 관련된 코드는 비동기적인 코드입니다.

* setTimeout : 사용자의 요청에 의해 특정 시간이 경과되기 전까지 어떤 함수의 실행을 보류

* addEventListener : 사용자의 직접적인 개입이 있을 때 어떤 함수를 실행하도록 대기

* XMLHttpRequest : 웹브라우저 자체가 아닌 별도의 대상에 무언가를 요청하고 그에 대한 응답이 왔을 때 비로소 어떤 함수를 실행하도록 함

### **비동기 작업의 동기적 표현 - Promise**

ES6의 Promise를 이용해서 비동기 제어를 할 수 있습니다.

new 연산자와 함께 호출한 Promise의 인자로 넘겨주는 콜백 함수는 호출할 때 바로 실행되지만 그 내부에 resolve 또는 reject 함수를 호출하는 구문이 있을 경우 둘 중 하나가 실행되기 전까지는 다음(then) 또는 오류구문(catch)으로 넘어가지 않습니다.

따라서, 비동기 작업이 완료될 때 비로소 resolve 또는 reject를 호출하는 방법으로 비동기 작업의 동기적 표현이 가능합니다.

```javascript
new Promise(function(resolve) { // (1)
   setTimeout(function() {
      var name = "에스프레소";
      console.log(name); 
      resolve(name); // (2)
   }, 500);
})

.then(function (prevName) {  // (3)
   return new Promise(function(resolve) {
      setTimeout(function() {
         var name = prevName + ", 아메리카노";
         console.log(name);
         resolve(name);
      }, 500);
   });
})

.then(function (prevName) {  // (4)
   return new Promise(function(resolve) {
      setTimeout(function() {
         var name = prevName + ", 카페라떼";
         console.log(name);
         resolve(name);
      }, 500);
   });
})

//  -- 결과 -- 
// 에스프레소
// 에스프레소, 아메리카노
// 에스프레소, 아메리카노, 카페라떼
```
1. 첫 번째 줄과 같이 선언할 경우 Promise 객체에 파라미터로 넘겨준 익명함수는 즉각 실행됩니다. new Promise()로 인스턴스를 생성하고- 이때 Promise() 파라미터에 작성한 function() {}을 실행합니다. function을 excuter(실행자)라고 합니다. 

* * excuter는 resolve와 reject라는 두 개의 함수를 매개변수로 받는 실행함수입니다.
excuter는 비동기 작업을 시작하고 모든 작업을 끝낸 후 해당 작업이 성공적으로 이행되었으면 resolve 함수를 호출하고, 중간에 오류가 발생했을 경우 reject 함수를 호출합니다.

2. excuter 안에서 name 변수에 "에스프레소"를 할당하고 console.log(name);를 실행하여 "에스프레소"가 출력됩니다.

3. 바로 밑의 resolve()는 함수를 호출하는 형태이지만 호출하지 않습니다. (아래에 이어서 설명합니다.)

4. 연결된 첫 then()의 핸들러 함수를 실행하지 않고 아래로 이동합니다.

5. 연결된 두 번째 then()의 핸들러 함수를 실행하지 않고 아래로 이동합니다.

6. 더 이상 처리할 코드가 없습니다.

7. 첫 번째의 excuter 블록의 resolve가 호출되어 첫 번째 then()의 핸들러 함수가 실행되며, 콘솔에 "에스프레소, 아메리카노" 가 출력됩니다.

- * then 메서드는 Promise 객체를 리턴하고 두 개의 콜백 함수를 인수로 받습니다.

   * then 메서드는 promise 객체를 리턴하고 인수로 받은 콜백 함수들의 리턴 값을 이어 받습니다. 따라서 chaning이 가능합니다.

8. 또 다시 excuter 블록의 resolve가 호출되어 두 번째 then()의 핸들러 함수가 실행되며, 콘솔에 "에스프레소, 아메리카노, 카페라떼" 가 출력됩니다.

### **비동기 작업의 동기적 표현 - Generator**

ES6의 Generator를 이용해서 비동기 제어를 할 수 있습니다.

7번째 줄의 '*'이 붙은 함수가 바로 Generator 함수입니다.

Generator 함수를 실행하면 Iterator가 반환되는데 Iterator는 next라는 메서드를 가지고 있습니다. 이 next 메서드를 호출하면 Generator 함수 내부에서 가장 먼저 등장하는 yield에서 함수의 실행을 멈춥니다. 이후 다시 next 메서드를 호출하면 앞서 멈췄던 부분부터 시작하여 그다음에 등장하는 yield에서 함수 실행을 멈춥니다. 

* 이터레이터는 "반복자"라는 의미로, 이터러블(Iterabe, 순회 가능한 자료구조)의 요소를 탐색하기 위한 포인터로서 next() 함수를 가지고 있는 객체입니다.

비동기 작업이 완료되는 시점마다 next 메서드를 호출해준다면 Generator 함수 내부의 소스가 위에서 아래로 순차적으로 진행됩니다.

```javascript
var addCoffee = function(prevName, name) { // (5)
   setTimeout(function() {
      coffeeMaker.next(prevName ? prevName + ", " + name : name);
   }, 500);
};

var coffeeGenerator = function* () { // (1)
   var espresso = yield addCoffee("", "에스프레소"); // (4)
   console.log(espresso); // (6)
   var americano = yield addCoffee(espresso, "아메리카노"); // (7)
   console.log(americano); // (8)
   var latte = yield addCoffee(americano, "카페라떼");
   console.log(latte);
};

var coffeeMaker = coffeeGenerator(); // (2)
coffeeMaker.next(); // (3)

//  -- 결과 -- 
// {value: undefined, done: false}
// 에스프레소
// 에스프레소, 아메리카노
// 에스프레소, 아메리카노, 카페라떼
```

1. (1)에서 표현식 형태로 제너레이터 함수를 정의합니다. var coffeeGenerator = function*(){} 형태에서 var coffeeGenerator를 제외하면 function*(){} 는 무명함수입니다. 생성한 함수를 변수에 할당해야 함수를 호출할 수 있으며, coffeeGenerator가 함수 이름이 됩니다. function*의 오른쪽에 함수 이름을 작성할 수 있지만, 외부에서 함수를 호출할 때는 coffeeGenerator()로 호출해야합니다.

2. (2)에서 coffeeGenerator()를 호출하면 제너레이터 오브젝트를 생성합니다. 이때 함수 블록의 코드를 실행하지 않고 생성한 제너레이터 오브젝트를 반환합니다.

3. (3) 제너레이터 오브젝트는 이터레이터 오브젝트입니다. 제너레이터 오브젝트의 next()를 호출하면 이터레이터 오브젝트와 같은 처리를 수행합니다. next()를 호출하면 coffeeGenerator 제너레이터 함수의 함수 블록을 수행합니다.

```javascript
var espresso = yield addCoffee("", "에스프레소");
console.log(espresso);
...
```
4. (4) 위의 두 줄은 제너레이터 함수 블록의 코드입니다. 함수 블록의 첫 줄부터 첫 번째 yield까지 수행합니다. yield 키워드는 오른쪽의 표현식을 평가하고, 평가 결과를 {value: undefined, done: false} 형태로 반환합니다.

* [returnValue] = yield [expression]
   * yield 키워드의 오른쪽 표현식(expression)은 선택으로, 표현식을 작성하면 이를 평가하고 평가 결과를 반환합니다. 표현식을 작성하지 않으면 undefined를 반환하며 yield의 표현식 결과를 왼쪽의 [returnValue]에 할당하지 않습니다. 제너레이터 오브젝트의 next()를 호출하면 next() 파라미터 값이 [returnValue]에 설정됩니다.

   * next()로 제너레이터 함수를 호출하면 yield 작성에 관계 없이 "{value:값, done:true/false}" 형태로 반환합니다.

   * yield를 수행하면 표현식 평가 결과가 value 값에 설정되고, yield를 수행하지 못하면 undefined가 설정됩니다.

   * yield를 수행하면 done 값에 false가 설정되고, yield를 수행하지 못하면 true가 설정됩니다.

   * 제너레이터 함수의 모든 yield 수행을 완료했는데, 다시 next()를 호출하면 수행할 yield가 없으므로 value 값에 undefined가 설정되고 done 값에 true가 설정됩니다.

5. (5) prevName에 빈 값이 들어가고 name에 "에스프레소가" 들어가므로 "에스프레소를" espresso에 설정하게됩니다.

6. (5)에서 coffeeMaker.next("에스프레소")를 호출하게 되어 첫 번째 yield 이후의 코드부터 두 번째 yield까지 실행합니다. 제너레이터 함수에서 반환된 결과를 espresso 변수에 할당하고 "에스프레소"를 출력합니다.

7. (7) prevName에는 "에스프레소"가, name에는 "아메리카노"가 할당되므로 (5)에서 "아메리카노, 에스프레소" 라는 값이 설정됩니다.

8. 이후는 위의 과정과 같습니다. addCoffee 함수의 setTimeout 안에서 next를 만나 이전 yield 이후의 코드부터 다음 yield 코드까지 실행합니다. 

```javascript 
// 콜백 예제를 이용한 제너레이터 (다른 ver.) *같이 풀어보실래요?*

function helloWorld_Hello() {
  setTimeout(() => {
    iterator.next('Hello');
  }, 1000);
}
 
function helloWorld_Space(text) {
  setTimeout(() => {
    iterator.next(text + ' ');
  }, 1000);
}
 
function helloWorld_World(text) {
  setTimeout(() => {
    iterator.next(text + 'World');
  }, 1000);
}
 
function* helloWorld() {
    const hello = yield helloWorld_Hello();
    const space = yield helloWorld_Space(hello);
    const world = yield helloWorld_World(space);
    console.log(world); // Hello World
    return;
}
const iterator = helloWorld();
iterator.next();

// -- 결과 --
// {value: undefined, done: false}
// Hello World
```

### **비동기 작업의 동기적 표현 - Promise + Async/await**

ES2017에서는 Async/await가 추가됐습니다. 

비동기 작업을 수행하고자 하는 암수 앞에 async를 표기하고, 함수 내부에서 실질적인 비동기 작업이 필요한 위치마다 await를 표기하는 것만으로 뒤의 내용을 promise로 자동 전환합니다.

해당 내용이 resolve된 이후에야 다음으로 진행합니다. 즉 Promise의 then과 흡사한 효과를 얻을 수 있습니다.

```javascript
var addCoffee = function(name) {
   return new Promise(function(resolve) {
      setTimeout(function() {
         resolve(name);
      }, 500);
   });
};

var coffeeMaker = async function() {
   var coffeeList = "";
   var _addCoffee = async function(name) { 
      coffeeList += (coffeeList ? ", " : "") + await addCoffee(name); 
   };

   await _addCoffee("에스프레소"); 
   console.log(coffeeList);
   await _addCoffee("아메리카노"); 
   console.log(coffeeList);
   await _addCoffee("카페라떼"); 
   console.log(coffeeList);
};

coffeeMaker();
```


