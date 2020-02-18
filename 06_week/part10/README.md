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
