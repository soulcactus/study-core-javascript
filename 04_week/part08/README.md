# 클로저

## 🧐 클로저의 활용 사례

지난 스터디에서 클로저의 의미와 작동 원리를 어느저

### 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때
다음 예제는 대표적인 콜백 함수 중 하나인 이벤트 리스너에 관한 예시입니다. 클로저의 '외부 데이터'에 주목하면서 흐름을 따라가보겠습니다.

```javascript
// 예제) 콜백 함수와 클로저(1)
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

(A)는 fruits의 개수만큼 실행되며, 그때마다 새로운 실행 컨텍스트가 활성화됩니다. (A)의 실행 종료 여부와 무관하게 클릭 이벤트에 의해 각 컨텍스트의 (B)가 실행될 때는 (B)의 outerEnvironmentReference가 (A)의 LexicalEnvironment를 참조합니다. 따라서 최소한 (B)함수가 참조할 예정인 변수 fruit에 대해서는 (A)가 종료된 후에도 GC 대상에서 제외되어 계속 참조 가능하게 됩니다. 

(B) 함수의 쓰임새가 콜백 함수에 국한되지 않는 경우라면 반복을 줄이기 위해 (B)를 외부로 분리하는 편이 나을 수 있습니다. 즉, fruit를 인자로 받아 출력하는 형태로 바꿔볼 수 있겠습니다.

```javascript
// 예제) 콜백 함수와 클로저(2)
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

위 예제에서는 공통 함수로 쓰고자 콜백 함수를 외부로 꺼내어 alertFruit라는 변수에 담았습니다. 이제 (alertFruit(fruits[1]);) 실행 시 정상적으로 alertFruit을 직접 실행할 수 있습니다. 마지막 줄에서는 정상적으로 'banana'에 대한 alert이 실행됩니다.

그런데 각 li를 클릭하면 클릭한 대상의 과일명이 아닌 [object MouseEvent]라는 값이 출력됩니다. 콜백 함수의 인자에 대한 제어권을 addEventListener가 가진 상태이며, addEventListener는 콜백 함수를 호출할 때 첫 번째 인자에 '이벤트 객체'를 주입하기 때문입니다. 이 문제는 bind 메서드를 활용하면 해결할 수 있습니다. 

```javascript
// 예제) 콜백 함수와 클로저(3)
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
다만 이렇게 bind 메서드를 사용하면 이벤트 객체가 인자로 넘어오는 순서가 바뀌는 점, 함수 내부에서의 this가 원래의 그것과 달라지는 점은 감안해야합니다.

이런 변경사항이 발생하지 않게끔 하면서 이슈를 해결하기 위해서는 bind 메서드가 아닌 고차함수를 활용하는 방법이 있습니다. 이는 함수형 프로그래밍에서 자주 쓰이는 방식이기도 합니다.

```javascript
// 예제) 콜백 함수와 클로저(4)
var fruits = ['apple', 'banana', 'peach'];
var $ul = document.createElement("ul");

var alertFruitBuilder = function(fruit) { 
    return function() { // 익명함수 반환. 기존의 alertFruit 함수.
        alert("your choice is " + fruit);
    };
};

fruits.forEach(function(fruit) { // 콜백함수(A)
    var $li = document.createElement("li");
    $li.innerText = fruit;
    // alertFruitBuilder 함수 실행 시 fruit 값을 인자로 전달.
    $li.addEventListener("click", alertFruitBuilder(fruit));
    $ul.appendChild($li);
});

document.body.appendChild($ul);
alertFruitBuilder(fruits); // your choice is banana
// 각 li 클릭 시 alert : your choice is apple/banana/peach 
```

4번째 줄의 alertFruitBuilder 함수 내부에서는 다시 익명함수를 반환하는데, 이 익명함수가 바로 기존의 alertFruit 함수입니다.

13번째 줄에서는 alertFruitBuilder 함수를 실행하면서 fruit 값을 인자로 전달했습니다. 그러면 이 함수의 실행 결과가 다시 함수가 되며, 이렇게 반환된 함수를 리스너에 콜백 함수로써 전달하게 됩니다.

이후 언젠가 클릭 이벤트가 발생하면 비로소 이 함수의 실행 컨텍스트가 열리면서 alertFruitBuilder의 인자로 넘어온 fruit를 outerEnvironmentReference에 의해 참조할 수 있게됩니다. 즉 alertFruitBuilder의 실행 결과로 반환된 함수에는 클로저가 존재합니다. 

지금까지 콜백 함수 내부에서 외부변수를 참조하기 위한 방법 세 가지를 살펴봤는데요.

1. 첫 번째 예제는 콜백 함수를 내부함수로 선언해서 외부변수를 직접 참조하는 방법으로, 클로저를 사용한 방법이었습니다.

2. 두 번째 예제에서는 bind를 활용했는데, bind 메서드로 값을 직접 넘겨준 덕분에 클로저는 발생하지 않게 된 반면 여러 가지 제약사항이 따르게 됐습니다. 

3. 세 번째 예제는 콜백 함수를 고차함수로 바꿔 클로저를 적극적으로 활용한 방안이었습니다. 

위 세 방법의 장단점을 각기 파악하고 구체적인 상황에 따라 어떤 방법을 도입히는 것이 가장 효과적일지 고민해보면 좋을 것 같습니다.

### 접근 권한 제어 (정보 은닉)

정보 은닉(information hiding)이란, 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화하여 모듈간의 결합도를 낮추고 유연성을 높이고자 하는 현대 프로그래밍 언어의 중요한 개념 중 하나입니다.

흔히 접근 권한에는 public, private, protected의 세 종류가 있습니다. 

* public은 외부에서 접근 가능한 것

* private는 내부에서만 사용하며 외부에 노출되지 않는 것

* protected는 상속받은 클래스 또는 같은 패키지에서만 접근 가능한 것

자바스크립트는 기본적으로 변수 자체에 이러한 접근 권한을 직접 부여하도록 설계돼 있지 않습니다.

그렇다고 접근 권한 제어가 불가능한 것은 아닌데요. 클로저를 이용하면 함수차원에서 public한 값과 private한 값을 구분하는 것이 가능합니다.

지난 클로저 개념파트에서 봤던 예제를 다시 보겠습니다.

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

outer 함수를 종료할 때 inner 함수를 반환함으로써 outer 함수의 지역변수인 a의 값을 외부에서도 읽을 수 있게 됐습니다. 이처럼 클로저를 이용하면 return을 활용하여 외부 스코프에서 함수 내부의 변수들 중 선택적으로 일부의 변수에 대한 접근 권한을 부여할 수 있습니다.

외부(전역 스코프)에서는 외부 공간에 노출돼 있는 outer라는 변수를 통해 outer 함수를 실행할 수는 있지만, outer 함수 내부에서는 어떠한 개입도 할 수 없습니다. 외부에서는 오직 outer 함수가 return한 정보에만 접근할 수 있습니다.

외부에 제공하고자 하는 정보들을 모아 return 하고, 내부에서만 사용할 정보들은 return하지 않는 것으로 접근 권한 제어가 가능한 것입니다. return한 변수들은 공개 멤버(public member), 그렇지 않은 변수들은 비공개 멤버(private member)가 됩니다.

밑에서 예제로 간단한 게임을 만들면서 접근 권한을 제어해 보겠습니다. 규칙은 다음과 같습니다.

* 각 턴마다 주사위를 굴려 나온 숫자(km)만큼 이동.

* 차량별로 연료량(fuel)과 연비(power)는 무작위로 생성.

* 남은 연료가 이동할 거리에 필요한 연료보다 부족하면 이동하지 못함.

* 모든 유저가 이동할 수 없는 턴에서 게임 종료.

* 게임 종료 시점에 가장 멀리 이동해 있는 사람이 승리.

```javascript
var car = {
    fuel : Math.ceil(Math.random() * 10 + 10), // 연료량
    power : Math.ceil(Math.random() * 3 + 2), // 연비
    moved : 0, // 총 이동거리
    run : function() {
        var km = Math.ceil(Math.random() * 6); // 주사위 숫자 (이동 거리)
        var wasteFuel = km / this.power;
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

car 변수에 객체를 직접 할당했습니다.

fuel과 power는 무작위로 생성하고, moved라는 프로퍼티에 총 이동거리를 부여했으며, run 메서드를 실행할 때마다 car 객체의 fuel, moved 값이 변하게 했습니다. 이러한 car 객체를 사람 수만큼 생성하여 각자의 턴에 run을 실행하면 게임을 즐길 수 있습니다.

모두가 run 메서드만 호출한다는 가정하에는 괜찮겠지만 자바스크립트를 다룰 줄 아는 사람이 참여할 경우 무작위로 정해지는 연료, 연비, 이동거리 등을 마음대로 바꿔 게임 결과를 조작할 수 있게되겠죠?

```javascript
car.fuel = 10000;
car.power = 100;
car.moved = 1000;
```

이런 식으로 마음대로 값을 바꿔버리면 일방적인 게임이 되어버립니다.

이렇게 값을 바꾸지 못하도록 클로저를 활용하여 방어할 수 있습니다. 즉, 객체가 아닌 함수로 만들고, 필요한 친구들만을 return하는 것입니다.

```javascript
var createCar = function() {
    var fuel = Math.ceil(Math.random() * 10 + 10);
    var power = Math.ceil(Math.random() * 3 + 2);
    var moved = 0;

    return {
        get moved() { // getter만 부여(읽기 전용 속성)
            return moved;
        },
        run : function() {
            var km = Math.ceil(Math.random() * 6);
            var wasteFuel = km / power;
            if (fuel < wasteFuel) {
                console.log("이동불가");
                return;
            }
            fuel -= wasteFuel;
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

run 메서드를 다른 내용으로 덮어씌우는 조작은 여전히 가능한 상태이긴 하지만 이전 예시 코드보다는 훨씬 안전한 코드가 됐습니다.

이런 조작까지 막기 위해서는 객체를 return하기 전에 미리 변경할 수 없게끔 조치를 취해야합니다.

```javascript
var createCar = function() {
  var fuel = Math.ceil(Math.random() * 10 + 10);
  var power = Math.ceil(Math.random() * 3 + 2);
  var moved = 0;

  var publicMembers = {
    get moved() {
      return moved;
    },
    run : function() {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / power;
      if (fuel < wasteFuel) {
        console.log("이동불가");
        return;
      }
      fuel -= wasteFuel; 
      moved += km;
      console.log(km + "km 이동 (총 " + moved + "km). 남은 연료 : " + fuel);
    }
  }
  Object.freeze(publicMembers); // 객체 동결(변경 불가)
  return publicMembers
};

var car = createCar();
```

Object.freeze 메서드를 사용하여 객체를 동결합니다. 동결된 객체는 더 이상 변경될 수 없습니다. 동결된 객체는 새로운 속성을 추가하거나 존재하는 속성을 제거하는 것을 방지하며 존재하는 속성의 불변성, 설정가능성, 작성 가능성이 변경되는 것을 방지하고, 존재하는 속성의 값이 변경되는 것도 방지합니다.

freeze()는 전달된 동일한 객체를 반환합니다. 이 정도면 충분히 안전한 객체가 되었습니다.

다시 정리해보자면 클로저를 활용해 접근권한을 제어하는 방법은 다음과 같습니다.

1. 함수에서 지역변수 및 내부함수 등을 생성합니다.

2. 외부에 접근권한을 주고자 하는 대상들로 구성된 참조형 데이터(대상이 여럿일 때는 객체 또는 배열, 하나일 때는 함수)를 return합니다. return한 변수들은 공개 멤버가 되고, 그렇지 않은 변수들은 비공개 멤버가 됩니다.


### 부분 적용 함수
부분 적용 함수란 n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 하는 함수입니다.

```javascript
// 예시) bind 메서드를 활용한 부분 적용 함수
var add = function() {
  var result = 0;
  for (var i = 0; i < arguments.length; i++) {
    result += arguments[i];
  }
  return result;
};

var addPartial = add.bind(null, 1, 2, 3, 4, 5); // 인자 5개를 미리 적용
console.log(addPartial(6, 7, 8, 9,10)); // 55
```

위 예제의 addPartail 함수는 인자 5개를 미리 적용하고, 추후 추가적으로 인자들을 전달하면 모든 인자를 모아 원래의 함수가 실행되는 부분 적용 함수입니다. add 함수는 this를 사용하지 않으므로 bind 메서드만으로도 문제 없이 구현됐습니다. 

다만, this의 값을 변경할 수밖에 없기 때문에 bind 메서드에서는 사용할 수 없을 것 같습니다. 범용성 측면에서 this에 관여하지 않는 별도의 부분 적용 함수가 있다면 더욱 좋겠죠?

```javascript
// 예시) 부분 적용 함수 구현(1)
var partial = function() {
    var originalPartialArgs = arguments;
    var func = originalPartialArgs[0];
    if (typeof func !== "function") {
        throw new Error("첫 번째 인자가 함수가 아닙니다.");
    }
    return function() {
        var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1); // [1, 2, 3, 4, 5]
        var restArgs = Array.prototype.slice.call(arguments); // [6, 7, 8, 9, 10]
        return func.apply(this, partialArgs.concat(restArgs)); // (2)
    };
};

var add = function() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
        result += arguments[i];
    }
    return result;
};

var addPartial = partial(add, 1, 2, 3, 4, 5); // (1)
console.log(addPartial(6, 7, 8, 9, 10)); // 55

var hwizzzang = {
    name : "휘쨩- ",
    greet : partial(function(prefix, suffix) {
        console.log("prefix : " + prefix); // prefix : 헥헥이  
        console.log("suffix : " + suffix); // suffix : 드디어 이해하다!
        return prefix + this.name + suffix;
    }, "헥헥이 ")
};

hwizzzang.greet("드디어 이해하다!");
```

(1)에서 첫 번째 인자에는 원본 함수를, 두 번째 인자 이후부터는 미리 적용할 인자들을 전달하고, (2) 반환할 함수(부분 적용 함수)에서는 다시 나머지 인자들을 받아 이들을 한데 모아(concat) 원본 함수를 호출(apply)합니다. 또한 실행 시점의 this를 그대로 반영함으로써 this에는 아무런 영향을 주지 않게 됐습니다.

* apply - fn.apply(thisArg, [argsArray]) : this 인자를 첫 번째 인자로 받고, 두 번째 인자로는 **배열**을 받습니다.

* call - fn.call(thisArg[, arg1[, arg2[, ...]]]) : this 인자를 첫 번째 인자로 받고, 두 번째 인자부터는 **배열이 아닌 각 인자**로 받습니다.

* bind - fn.bind(thisArg[, arg1[, arg2[, ...]]]) : call와 인자 작성법은 같으나, apply/call 과 달리 **바로 메소드가 실행되지 않음**. **thisArg를 바인딩하는 역할**만 합니다.

* call, apply는 함수를 실행하여 그 함수의 값을, bind는 지정한 객체의 새로운 함수를 만듭니다. 쉽게 말해, call/apply는 그냥 함수가 실행되도록 도와주는 것이고, bind는 새로운 함수를 만들어줍니디.

* Array.prototype.slice.call(arguments) : arguments는 전달인자들을 배열처엄 인덱스로 접근할 수 있지만, 실제 배열이 아니기 때문에 push, forEach, indexOf 등과 같은 메서드는 존재하지 않으며 사용할 수 없습니다. 이러한 형태를 띄는 것이 Array-like objects입니다. 이는 배열 형태로 바꾸어 사용할 수 있는데 Array.prototype.slice.call(arguments); 등과 같이 slice와 call을 활용할 수 있습니다.

  ```javascript
  function list1() {
    return arguments;
  }
  
  list1(1, 2, 3); // [1, 2, 3, callee: function, Symbol(Symbol.iterator):function]

  function list2() {
    return Array.prototype.slice.call(arguments);
  }

  list2(1, 2, 3); // [1, 2, 3]

  function func(arg1) {
    var rest = Array.prototype.slice.call(arguments, 1);
    console.log(rest);
  }

  func(1, 2, 3, 4); // [2, 3, 4]
  ```

위 예제 코드는 부분 적용 함수에 넘길 인자를 반드시 앞에서부터 차례로 전달할 수밖에 없다는 점이 조금 아쉬운데요. 인자들을 원하는 위치에 미리 넣어놓고 나중에 빈 자리에 인자를 채워넣어 실행할 수 있게끔 코드를 수정해보겠습니다.

```javascript
// 예시) 부분 적용 함수 구현(2)

Object.defineProperty(window, "_", {
  value : "EMPTY_SPACE",
  writeable : false, // 쓰기 불가능
  configurable : false, // 구성 불가능
  enumerable : false // 열거, 셈 불가능
});

var partial = function() {
    var originalPartialArgs = arguments;
    var func = originalPartialArgs[0];
    
    if(typeof func !== "function") {
        throw new Error("첫 번째 인자가 함수가 아님");
    }

    return function() {
        var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
        var restArgs = Array.prototype.slice.call(arguments);

        for (var i = 0; i < partialArgs.length; i++) {
            if(partialArgs[i] === _) {
              partialArgs[i] = restArgs.shift();
            } 
        }

        return func.apply(this, partialArgs.concat(restArgs));
    };
};

var add = function() {
    var result = 0;
    for (var i = 0; i< arguments.length; i++) {
        result += arguments[i];
    }
    return result;
};

var addPartial = partial(add, 1, 2, _, 4, 5, _, _, 8, 9); // (1)
console.log(addPartial(3, 6, 7, 10));

var hwizzzang = {
    name : "휘쨩- ",
    greet : partial(function(prefix, suffix) {
        console.log("prefix : " + prefix);   
        console.log("suffix : " + suffix);
        return prefix + this.name + suffix;
    }, "헥헥이 ")
};

hwizzzang.greet("드디어 이해를!?"); // 헥헥이, 휘쨩- 드디어 이해하다!
```
* Object.defineProperty() : 정적 메서드는 객체에 직접 새로운 속성을 정의하거나 이미 존재하는 속성을 수정한 후, 그 객체를 반환합니다.

이번에는 '비워놓음'을 표시하기 위해 미리 전역객체에 _라는 프로퍼티를 준비하면서 삭제 변경 등의 접근에 대한 방어 차원에서 여러 가지 프로퍼티 속성을 설정했습니다.

(1) 처음에 넘겨준 인자들 중 _로 비워놓은 공간마다 나중에 넘어온 인자들이 차례대로 끼워넣어지도록 구현됐습니다. 부분 적용 함수를 만들 때 미리 실행할 함수의 모든 인자 개수를 맞춰 빈 공간을 확보하지 않아도 되며 최종 실행 시 인자 개수가 많은 적든 잘 실행됩니다.

이전 예제와 위에서 본 예제의 부분 적용 함수들은 모두 클로저를 핵심 기법으로 사용하였습니다. 미리 일부 인자를 넘겨두어 기억하게끔 하고 추후 필요한 시점에 기억했던 인자들까지 함께 실행하게 한다는 개념 자체가 클로저의 정의에 정확히 부합합니다.

#### + 참고
ES5 환경에서는 _를 '비워놓음'으로 사용하기 위해 어쩔 수 없이 전역 공간을 침범했는데요. ES6에서는 Symbol.for를 활용하면 훨씬 좋습니다. Symbol.for 메서드는 전역 심볼공간에 인자로 넘어온 문자열이 이미 있으면 해당 값을 참조하고, 선언돼 있지 않으면 새로 만드는 방식으로 어디서든 접근 가능한 상수를 만들고자 할 때 적합합니다.

```javascript
(function() {
  // 기존 전역 심볼공간에 EMPTY_SPACE라는 문자열을 가진 심볼이 없으므로 새로 생성
  var EmptySpace = Symbol.for("EMPTY_SPACE");
  console.log(EmptySpace);
})();

(function() {
  // 기존 전역 심볼공간에 EMPTY_SPACE라는 문자열의 심볼이 있으므로 해당 값을 참조
  console.log(Symbol.for("EMPTY_SPACE"));
})();
```
이 Symbol.for를 이용하면 위의 예제(부분 적용 함수 구현(2))을 다음과 같이 바꿀 수 있습니다.

```javascript
// ... 앞부분의 Object.defineProperty(window, "_", {...}) 삭제 

var partial = function() {
    // ... 생략 

    return function() {
        // ... 생략 

        for (var i = 0; i < partialArgs.length; i++) {
            if(partialArgs[i] === Symbol.for("EMPTY_SPACE")) { // 바뀐 부분!!
              partialArgs[i] = restArgs.shift();
            } 
        }

        return func.apply(this, partialArgs.concat(restArgs));
    };
};

// ... 생략

var _ = Symbol.for("EMPTY_SPACE"); // 추가된 부분!!
var addPartial = partial(add, 1, 2, _, 4, 5, _, _, 8, 9);
console.log(addPartial(3, 6, 7, 10));
```

다음 예제에서는 디바운스를 소개하겠습니다.

디바운스란 **짧은 시간 동안 동일한 이벤트가 많이 발생할 경우 이를 전부 처리하지 않고 처음 또는 마지막에 발생한 이벤트에 대해 한 번만 처리**하는 것으로, 프론트엔드 성능 최적화에 큰 도움을 주는 기능 중 하나입니다. scroll, wheel, mousemove, resize 등에 적용하기 좋습니다.

```javascript
// 예시) 부분 적용 함수 - 디바운스

var debounce = function(eventName, func, wait) { // (1)
    var timeoutId = null; // (2)
    return function(event) { // (3)
        var self = this; // (4)
        console.log(eventName, "event 발생"); 
        clearTimeout(timeoutId); // (5) (7)
        timeoutId = setTimeout(func.bind(self, event), wait); // (6) (8)
    };
};

var moveHandler = function(e) {
    console.log("move event 처리");
}

var wheelHandler = function(e) {
    console.log("wheel event 처리");
}

document.body.addEventListener("mousemove", debounce("move", moveHandler, 1000));
document.body.addEventListener("mousewheel", debounce("wheel", wheelHandler, 1000));
```

(1) var debounce = function(eventName, func, wait)
* 디바운스 함수는 출력 용도로 지정한 eventName, 실행할 함수 func, 마지막으로 발생한 이벤트인지 여부를 판단하기 위한 대기시간 wait(ms)를 받습니다.

(2) var timeoutId = null;
* 내부에서는 timeoutId 변수를 생성하고

(3) return function(event) { ... }
* 클로저로 EventListener에 의해 호출될 함수를 반환합니다.

(4) var self = this;
* 반환될 함수 내부에서는, setTimeout을 사용하기 위해 this를 별도의 변수에 담고

(5) clearTimeout(timeoutId);
* 해당 줄에서 무조건 대기큐를 초기화하게 했습니다. 

(6) timeoutId = setTimeout(func.bind(self, event), wait);
* 마지막으로 7번째 줄에서 setTimeout으로 wait 시간만큼 지연시킨 후, 원래의 func를 호출하는 형태입니다. 

(7) clearTimeout(timeoutId);
*  이제 최초 event가 발생하면 이번에는 6번째 줄에 의해 앞서 저장했던 대기열을 초기화하고

(8) timeoutId = setTimeout(func.bind(self, event), wait); 
* 다시 7번째 줄에서 새로운 대기열을 등록합니다.

결국 각 이벤트가 바로 이전 이벤트로부터 wait 시간 이내에 발생하는 한, 마지막에 발생한 이벤트만이 초기화되지 않고 무사히 실행될 것입니다.

위 예제의 디바운스 함수에서 클로저로 처리되는 변수에는 eventName, func, wait, timeoutId가 있습니다.


### 커링 함수

커링 함수란 **여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 것**을 말합니다.

앞에서 살펴본 부분 적용 함수와 기본적인 맥락은 일치하지만 몇 가지 다른 점이 있습니다.

부분 적용 함수는 여러 개의 인자를 전달할 수 있고, 실행 결과를 재실행할 때 원본 함수가 무조건 실행되는 반면

커링은 **한 번에 하나의 인자만 전달**하는 것을 원칙으로 합니다. 또한 **중간 과정상의 함수를 실행한 결과는 그다음 인자를 받기 위해 대기만 할 뿐, 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않습니다.** 밑의 예제를 보겠습니다.

```javascript
var curry3 = function(func) {
  return function(a) {
    return function(b) {
      return func(a, b);
    };
  };
};

var getMaxWith10 = curry3(Math.max)(10);
console.log(getMaxWith10(8)); // 10
console.log(getMaxWith10(25)); // 25

var getMinWith10 = curry3(Math.min)(10);
console.log(getMinWith10(8)); // 8
console.log(getMinWith10(25)); // 10
```

부분 적용 함수와 달리 커링 함수는 필요한 상황에 직접 만들어 쓰기 용이합니다. 필요한 인자 개수만큼 함수를 만들어 계속 리턴해주다가 마지막에 조합해서 리턴해주면 되기 때문이죠. 다만, 인자가 많아질수록 가독성이 떨어진다는 단점이 있습니다. 밑의 예제처럼 말이죠.

```javascript
var curry5 = function(func) {
  return function(a) {
    return function(b) {
      return function(c) {
        return function(d) {
          return function(e) {
            return func(a, b, c, d, e);
          };
        };
      };
    };
  };
};

var getMax = curry5(Math.max);
console.log(getMax(1)(2)(3)(4)(5));
```

5개만 받아서 처리했음에도 이를 표현하기 위해 무려 13줄이나 소모했습니다. 다행히 ES6에서는 화살표 함수를 써서 같은 내용을 단 한 줄에 표기할 수 있습니다.

```javascript
var curry5 = func => a => b => c => d => e => func(a, b, c, d, e);
```

화살표 순서에 따라 함수에 값을 차례로 넘겨주면 마지막에 func가 호출될 거라는 흐름이 한눈에 파악됩니다. 

각 단계에서 받은 인자들을 모두 **마지막 단계에서 참조**할 것이므로 GC되지 않고 메모리에 차곡차곡 쌓였다가, 마지막 호출로 실행 컨텍스트가 종료된 후에야 비로소 한꺼번에 GC의 수거 대상이 됩니다.

이 커링 함수가 유용한 경우가 있는데요. 커링 함수는 당장 필요한 정보만 받아서 전달하고, 또 필요한 정보가 들어오면 전달하는 식으로 하면 결국 마지막 인자가 넘어갈 때까지 함수 실행을 미루는 셈이 됩니다. 이를 함수형 프로그래밍에서는 **지연 실행(lazy execution)** 이라고 칭합니다.

**원하는 시점까지 지연시켰다가 실행하는 상황**이거나, 프로젝트 내에서 자주 쓰이는 **함수의 매개변수가 항상 비슷하고 일부만 바뀌는 경우**에도 적절하게 쓰일 수 있습니다. 예를 들어, 다음 코드를 보겠습니다.

```javascript
var getInformation = function(baseUrl) { // 서버에 요청할 주소의 기본 URL
  return function(path) { // path 값
    return function(id) { //id 값
      return fetch(baseUrl + path + "/" + id); // 실제 서버에 정보 요청
    };
  };
};

// ES6
var getInformation = baseUrl => path => id => fetch(baseUrl + path + "/" + id);
```

HTML5의 fetch 함수는 url을 받아 해당 url에 HTTP 요청을 합니다. 보통 REST API를 이용할 경우 baseUrl은 몇 개로 고정되지만 나머지 path나 id 값은 매우 많을 수 있습니다. 서버에 정보를 요청할 필요가 있을 때마다 매번 baseUrl부터 전부 기입하는 것보다는 공통적인 요소는 먼저 기억시켜두고 특정한 값(id)만으로 서버 요청을 수행하는 함수를 만들어두는 편이 개발 효율성이나 가독성 측면에서 더 좋을 것입니다.

```javascript
var imageUrl = "http://imageAddress.com/";
var productUrl = "http://productAddress.com/";

// 이미지 타입별 요청 함수 준비
var getImage = getInformation(imageUrl); // http://imageAddress.com/
var getEmoticon = getImage("emoticon"); // http://imageAddress.com/emoticon
var getIcon = getImage("icon"); // http://imageAddress.com/icon

// 제품 타입별 요청 함수 준비
var getProduct = getInformation(productUrl); //http://productAddress.com/
var getFruit = getProduct("fruit"); // http://productAddress.com/fruit
var getVegetable = getProduct("vegetable"); //http://productAddress.com/vegetable

// 실제 요청
var emoticon = getEmoticon(100); // http://imageAddress.com/emoticon/100
var icon = getIcon(205) // http://imageAddress.com/icon/205
var fruit = getFruit(300)  // http://productAddress.com/fruit/300
var vegetable = getVegetable(456)  // http://productAddress.com/vegetable/456
```

Flux 아키텍처의 구현체 중 하나인 Redux의 미들웨어를 예를 들면 다음과 같습니다.

```javascript
// Redux Middleware "Logger"
const logger = store => next => action => {
  console.log("dispatching", action);
  console.log("next state", store.getState());
  return next(action);
}
// Redux Middleware "thuk"
const thunk = store => next => action => {
  return typeof action === "function"
  ? action(dispatch, store.getState)
  : next(action);
}
```
위 코드의 두 미들웨어는 공통적으로 store, next, action 순서로 인자를 받습니다. 이 중 store는 프로젝트 내에서 한 번 생성된 이후로는 바뀌지 않는 속성입니다. dispatch의 의미를 가지는 next 역시 마찬가지지만, action의 경우는 매번 달라집니다. 

store와 next 값이 결정되면 Redux 내부에서 logger 또는 thunk에 store, next를 미리 넘겨서 반환된 함수를 저장시켜놓고, 이후에는 action만 받아 처리할 수 있게끔 한 것입니다.

## 정리
클로저란 어떤 함수에서 선언한 변수를 참조하는 내부함수를 외부로 전달할 경우, 함수의 실행 컨텍스트가 종료된 후에도 해당 변수가 사라지지 않는 현상을 말합니다.

내부 함수를 외부로 전달하는 방법에는 함수를 return하는 경우 외에, 콜백으로 전달하는 경우도 포함됩니다.

클로저는 그 본질이 메모리를 계속 차지하는 친구이므로 사용하지 않게 된 클로저에는 메모리를 차지하지 않도록 관리해줄 필요가 있습니다.