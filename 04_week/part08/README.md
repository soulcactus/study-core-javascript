# 클로저

## 🧐 클로저의 활용 사례

### 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때
다음 예제는 대표적인 콜백 함수 중 하나인 이벤트 리스너에 관한 예시입니다. 클로저의 '외부 데이터'에 주목하면서 흐름을 따라가봅시다.

```javascript
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement("ul");

fruits.forEach(function(fruit) { // 콜백함수(A)
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", function() { // 콜백 함수(B)
    alert("your choice is " + fruit);
  });
  $ul.appendChild($li);
});

document.body.appendChild($ul);
```

위 예제에서는 fruits 변수를 순회하며 li를 생성하고, 각 li를 클릭하면 해당 리스너에 기억된 콜백 함수를 실행하게 합니다.

4번째 줄의 forEach 메서드에 넘겨준 익명의 콜백함수(A)는 그 내부에서 외부 변수를 사용하지 않고 있으므로 클로저가 없지만, 7번째 줄의 addEventListener에 넘겨준 콜백 함수(B)에는 fruit 라는 외부 변수를 참조하고 있으므로 클로저가 있습니다. 

(A)는 fruits의 개수만큼 실행되며, 그때마다 새로운 실행 컨텍스트가 활성화됩니다. (A)의 실행 종료 여부와 무관하게 클릭 이벤트에 의해 각 컨텍스트의 (B)가 실행될 때는 (B)의 outerEnvironmentReference가 (A)의 LexicalEnvironment를 참조하게 될 것입니다. 따라서 최소한 (B)함수가 참조할 예정인 변수 fruit에 대해서는 (A)가 종료된 후에도 GC 대상에서 제외되어 계속 참조 가능하게 됩니다. 

(B) 함수의 쓰임새가 콜백 함수에 국한되지 않는 경우라면 반복을 줄이기 위해 (B)를 외부로 분리하는 편이 나을 수 있습니다. 즉,  fruit를 인자로 받아 출력하는 형태로 말이죠. 밑에서 그렇게 바꾸어보도록 하겠습니다.

```javascript
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement("ul");

var alertFruit = function(fruit) {
  alert("your choice is " + fruit);
};

fruits.forEach(function(fruit) { // 콜백함수(A)
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruit);
  $ul.appendChild($li);
});

document.body.appendChild($ul);
alertFruit(fruits[1]); // your choice is banana
// 각 li 클릭 시 alert : your choice is [object MouseEvent]
```

위 예제에서는 공통 함수로 쓰고자 콜백 함수를 외부로 꺼내어 alertFruit라는 변수에 담았습니다. 이제 (alertFruit(fruits[1]);) 실행 시 정상적으로 alertFruit을 직접 실행할 수 있습니다. 마지막 줄에서는 정상적으로 'banana'에 대한 alert이 실행됩니다. 그런데 각 li를 클릭하면 클릭한 대상의 과일명이 아닌 [object MouseEvent]라는 값이 출력됩니다. 콜백 함수의 인자에 대한 제어권을 addEventListener가 가진 상태이며, addEventListener는 콜백 함수를 호출할 때 첫 번째 인자에 '이벤트 객체'를 주입하기 때문입니다. 이 문제는 bind 메서드를 활용하면 해결할 수 있습니다. 

```javascript
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement("ul");

var alertFruit = function(fruit) {
  alert("your choice is " + fruit);
};

fruits.forEach(function(fruit) { // 콜백함수(A)
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruit.bind(null, fruit)); // bind 메서드 활용
  $ul.appendChild($li);
});

document.body.appendChild($ul);
alertFruit(fruits[1]); // your choice is banana
// 각 li 클릭 시 alert : your choice is apple/banana/peach 
```
다만 이렇게 bind 메서드를 사용하면 이벤트 객체가 인자로 넘어오는 순서가 바뀌는 점, 함수 내부에서의 this가 원래의 그것과 달라지는 점은 감안해야합니다. 이런 변경사항이 발생하지 않게끔 하면서 이슈를 해결하기 위해서는 bine 메서드가 아닌 고차함수를 활용하는 방법이 있습니다. 이는 함수형 프로그래밍에서 자주 쓰이는 방식이기도 합니다.

```javascript
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement("ul");

var alertFruitBuilder = function(fruit) { 
  return function() { // 익명함수 반환. 기존의 alertFruit함수.
    alert("your choice is " + fruit);
  };
};

fruits.forEach(function(fruit) { // 콜백함수(A)
  var $li = document.createElement("li");
  $li.innerText = fruit;
  $li.addEventListener("click", alertFruitBuilder(fruit)); // alertFruitBuilder 함수 실행 시 fruit 값을 인자로 전달.
  $ul.appendChild($li);
});

document.body.appendChild($ul);
alertFruitBuilder(fruits); // your choice is banana
// 각 li 클릭 시 alert : your choice is apple/banana/peach 
```

4번째 줄의 alertFruitBuilder 함수 내부에서는 다시 익명함수를 반환하는데, 이 익명함수가 바로 기존의 alertFruit 함수입니다. 
13번째 줄에서는 alertFruitBuilder 함수를 실행하면서 fruit 값을 인자로 전달했습니다. 그러면 이 함수의 실행 결과가 다시 함수가 되며, 이렇게 반환된 함수를 리스너에 콜백 함수로써 전달하게 됩니다. 이후 언젠가 클릭 이벤트가 발생하면 비로소 이 함수의 실행 컨텍스트가 열리면서 alertFruitBuilder의 인자로 넘어온 fruit를 outerEnvironmentReference에 의해 참조할 수 있게됩니다. 즉 alertFruitBuilder의 실행 결과로 반환된 함수에는 클로저가 존재합니다. 

지금까지 콜백 함수 내부에서 외부변수를 참조하기 위한 방법 세 가지를 살펴봤는데요.

1. 첫 번째 예제는 콜백 함수를 내부함수로 선언해서 외부변수를 직접 참조하는 방법으로, 클로저를 사용한 방법이었습니다.
2. 두 번째 예제에서는 bind를 활용했는데, bind 메서드로 값을 직접 넘겨준 덕분에 클로저는 발생하지 않게 된 반면 여러 가지 제약사항이 따르게 됐습니다. 
3. 세 번째 예제는 콜백 함수를 고차함수로 바꿔 클로저를 적극적으로 활용한 방안이었습니다. 

위 세 방법의 장단점을 각기 파악하고 구체적인 상황에 따라 어떤 방법을 도입히는 것이 가장 효과적일지 고민해보면 좋을 것 같습니다.

### 접근 권한 제어 (정보 은닉)

정보 은닉이란, 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화하여 모듈간의 결합도를 낮추고 유연성을 높이고자 하는 현대 프로그래밍 언어의 중요한 개념 중 하나입니다.

흔히 접근 권한에는 public, private, protected의 세 종류가 있습니다. 각 단어의 의미 그대로, public은 외부에서 접근 가능한 것, private는 내부에서만 사용하며 외부에 노출되지 않는 것을 의미합니다.

자바스크립트는 기본적으로 변수 자체에 이러한 접근 권한을 직접 부여하도록 설계돼 있지 않습니다. 그렇다고 접근 권한 제어가 불가능한 것은 아닌데요. 클로저를 이용하면 함수 차원에서 public한 값과 private한 값을 구분하는 것이 가능합니다.

밑 예제를 보겠습니다.

```javascript
var outer = function() {
  var a = 1;
  var inner = function() {
    return ++a;
  };
  return inner;
};
var outer2 = outer();
console.log(outer2());
console.log(outer2());
```

outer 함수를 종료할 때 inner 함수를 반환함으로써 outer 함수의 지역변수인 a의 값을 외부에서도 읽을 수 있게 됐습니다. 이처럼 클로저를 활용하면 외부 스코프에서 함수 내부의 변수들 중 선택적으로 일부의 변수에 대한 접근 권한을 부여할 수 있습니다. 바로 return을 활용해서 말입니다.

outer 함수는 외부(전역 스코프)로부터 철저하게 격리된 닫힌 공간입니다. 외부에서는 외부 공간에 노출돼 있는 outer라는 변수를 통해 outer 함수를 실행할 수는 있지만, outer 함수 내부에서는 어떠한 개입도 할 수 없습니다. 외부에서는 오직 outer 함수가 return한 정보에만 접근할 수 있습니다. return 값이 외부에 정보를 제공하는 유일한 수단인 것입니다.

외부에 제공하고자 하는 정보들을 모아 return 하고, 내부에서만 사용할 정보들은 return하지 않는 것으로 접근 권한 제어가 가능한 것입니다. return한 변수들은 공개멤버(public member)가 되고, 그렇지 않은 변수들은 비공개 멤버(private member)가 되는 것이겠죠.

밑에서 간단한 게임을 만들면서 접근 권한을 제어해 보겠습니다. 규칙은 다음과 같습니다.

* 각 턴마다 주사위를 굴려 나온 숫자(km)만큼 이동.

* 차량별로 연료량(fuel)과 연비(power)는 무작위로 생성됨.

* 남은 연료가 이동할 거리에 필요한 연료보다 부족하면 이동하지 못함.

* 모든 유저가 이동할 수 없는 턴에 게임이 종료됨

* 게임 종료 시점에 가장 멀리 이동해 있는 사람이 승리.

```javascript
var car = {
  fuel : Math.ceil(Math.random() * 10 + 10), // 연료량
  power : Math.ceil(Math.random() * 3 + 2), // 연비
  moved : 0, // 총 이동거리
  run : function() {
    var km = Math.ceil(Math.random() * 6); // 주사위 굴려 나온 숫자 (이동하는 거리)
    var wasteFuel = km / this.power; // 
    if (this.fuel < wasteFuel) {
      console.log("이동불가");
      return;
    }
    this.fuel -= wasteFuel; // 남은 연료가 이동할 거리에 필요한 연료보다 부족하면 이동 못함
    this.moved += km;
    console.log(km + "km 이동 (총 " + this.moved = "km)");
  }
};
```

car 변수에 객체를 직접 할당했습니다. fuel과 power는 무작위로 생성하고, moved라는 프로퍼티에 총 이동거리를 부여했으며, run 메서드를 실행할 때마다 car 객체의 fuel, moved 값이 변하게 했습니다. 이러한 car 객체를 사람 수만큼 생성하여 각자의 턴에 run을 실행하면 게임을 즐길 수 있습니다.

모두가 run 메서드만 호출한다는 가정하에는 이 정도만으로도 충분하겠지만, 자바스크립트를 어느 정도라도 다룰 줄 알고 승부욕 강한 사람이 참여할 경우 무작위로 정해지는 연료, 연비, 이동거리 등을 마음대로 바꿔 게임 결과가 달라질 수 있습니다. 

```javascript
car.fuel = 10000;
car.power = 100;
car.moved = 1000;
```
이런 식으로 마음대로 값을 바꿔버리면 일방적인 게임이 되고 맙니다. 이렇게 값을 바꾸지 못하도록 클로저를 활용하여 방어할 수 있습니다. 즉, 객체가 아닌 함수로 만들고, 필요한 친구들만을 return하는 것입니다.

```javascript
var createCar = function() {
  var fuel = Math.ceil(Math.random() * 10 + 10); // 연료량
  var power = Math.ceil(Math.random() * 3 + 2); // 연비
  var moved = 0; // 총 이동거리

  return {
    get moved() { // getter만을 부여(읽기 전용 속성)
      return moved;
    },
    run : function() {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / power;
      if (fuel < wasteFuel) {
        console.log("이동불가");
        return;
      }
      fuel -= wasteFuel; // 남은 연료가 이동할 거리에 필요한 연료보다 부족하면 이동 못함
      moved += km;
      console.log(km + "km 이동 (총 " + moved + "km). 남은 연료 : " + fuel);
    }
  }
};

var car = createCar();
```

이번에는 createCar라는 함수를 실행함으로써 객체를 생성하게 했습니다. fuel, power 변수는 비공개 멤버로 지정해 외부에서의 접근을 제한했고, moved 변수는 getter만을 부여함으로써 읽기 전용 속성을 부여했습니다. 

이제 외부에서는 오직 run 메서드를 실행하는 것과 현재의 moved 값을 확인하는 두 가지 동작만 할 수 있습니다. 다음과 같이 값을 변경하려고 하여도 대부분 실패하게 됩니다.

```javascript
car.run();
console.log(car.moved);
console.log(car.fuel); // undefined
console.log(car.power); // undefinedt
car.fuel = 1000;
console.log(car.fuel); // 1000;
car.run(); 

car.power = 1000;
console.log(car.power); // 1000;
car.run(); 

car.moved = 1000;
console.log(car.moved); // 1000;
car.run(); 
```

run 메서드를 다른 내용으로 덮어씌우는 어뷰징은 여전히 가능한 상태이긴 하지만 이전 예시 코드보다는 훨씬 안전한 코드가 됐습니다. 이런 어뷰징까지 막기 위해서는 객체를 return하기 전에 미리 변경할 수 없게끔 조치를 취해야합니다.

```javascript
var createCar = function() {
  var fuel = Math.ceil(Math.random() * 10 + 10); // 연료량
  var power = Math.ceil(Math.random() * 3 + 2); // 연비
  var moved = 0; // 총 이동거리

  var publicMembers = {
    get moved() { // getter만을 부여(읽기 전용 속성)
      return moved;
    },
    run : function() {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / power;
      if (fuel < wasteFuel) {
        console.log("이동불가");
        return;
      }
      fuel -= wasteFuel; // 남은 연료가 이동할 거리에 필요한 연료보다 부족하면 이동 못함
      moved += km;
      console.log(km + "km 이동 (총 " + moved + "km). 남은 연료 : " + fuel);
    }
  }
  Object.freeze(publicMembers);
  return publicMembers
};

var car = createCar();
```

Object.freeze 메서드를 사용하여 객체를 동결합니다. 동결된 객체는 더 이상 변경될 수 없습니다. 즉 동결된 객체는 새로운 속성을 추가하거나 존재하는 속성을 제거하는 것을 방지하며 존재하는 속성의 불변성, 설정가능성, 작성 가능성이 변경되는 것을 방지하고, 존재하는 속성의 값이 변경되는 것도 방지합니다. freeze()는 전달된 동일한 객체를 반환합니다. 이 정도면 충분히 안전한 객체가 되었습니다. 이쯤에서 정리하고 다음으로 넘어가겠습니다.

클로저를 활용해 접근권한을 제어하는 방법은 다음과 같습니다.

1. 함수에서 지역변수 및 내부함수 등을 생성합니다.

2. 외부에 접근권한을 주고자 하는 대상들로 구성된 참조형 데이터(대상이 여럿일 때는 객체 또는 배열, 하나일 때는 함수)를 return합니다. return한 변수들은 공개 멤버가 되고, 그렇지 않은 변수들은 비공개 멤버가 됩니다.


### 부분 적용 함수
부분 적용 함수란 n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 하는 함수입니다.

```javascript
var add = function() {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};

var addPartial = add.bind(null, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9,10)); // 55
```

위 예제의 addPartail 함수는 인자 5개를 미리 적용하고, 추후 추가적으로 인자들을 전달하면 모든 인자를 모아 원래의 함수가 실행되는 부분 적용 함수입니다. add 함수는 this를 사용하지 않으므로 bind 메서드만으로도 문제 없이 구현됐습니다. 

그러나 this의 값을 변경할 수밖에 없기 때문에 메서드에서는 사용할 수 없을 것 같네요. this에 관여하지 않는 별도의 부분 적용 함수가 있다면 범용성 측면에서 더욱 좋겠습니다.

```javascript
// 흠,.,,,,,,,,,,
var partial = function() {
  var originalPartialArgs = arguments;
  var func = originalPartialArgs[0];
  if (typeof func !== "function") {
    throw new Error("첫 번째 인자가 함수가 아닙니다.");
  }
  return function() {
    // 함수.call(지정할 객체명, 전달할 매개변수)
    var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
    var restArgs = Array.prototype.slice.all(arguments);
    // fun.apply([thisObj[,argArray]])
    // 가져다쓸메소드.apply([현재객체로사용될객체[,함수에 전달될 인수 집합]])
    //함수.apply(지정할 객체명, [전달할 매개변수])
    return func.apply(this, partialArgs.concat(restArgs));
  };
};

var add = function() {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};

var addPartial = partial(add, 1, 2, 3, 4, 5);
console.log(addPartial(6, 7, 8, 9, 10)); // 55

var dog = {
  name : "강아지",
  greet : partial(function(prefix, suffix) {
    return prefix + this.name + suffix;
  }, "왈왈, ")
};

dog.greet("입니다");
```





