# 실행 컨텍스트와 호이스팅

## 🤘 목표

-   [ ]

####

## 📄 실행 컨텍스트

실행 컨텍스트는 실행 가능한 코드를 형상화하고 구분하는 추상적인 개념이지만 물리적으로는 객체의 형태를 가지며 3가지 프로퍼티를 소유합니다.(두 가지 의미 모두 포함)

**하지만 이 글에서는 명확한 설명을 위해 실행 컨텍스트(객체)와 실행 컨텍스트 환경(추상적인 개념)을 구분해 설명하도록 하겠습니다.**

실행 컨텍스트란 *실행할 코드에 제공할 환경 정보들을 모아놓은 객체*로 자바스크립트의 동적 언어로서의 성격을 가장 잘 파악할 수 있는 개념입니다.
자바스크립트는 어떤 실행 컨텍스트 환경이 활성화되는 시점에 선언된 변수를 위로 끌어올리고(호이스팅), 외부 환경 정보를 구성하고, this 값을 설정하는 등의 동작을 수행하는데(실행 컨텍스트 _객체_ 생성),
이로 인해 다른 언어에서는 발견할 수 없는 특이한 현상들이 발생합니다.

1. 실행 컨텍스트 환경 개념

    실행 가능한 코드를 형상화하고 구분하는 추상적인 개념으로, 즉 **실행 가능한 자바스크립트 코드 블록이 실행되는 환경**이라고 할 수 있습니다.
    전역 코드, eval 함수, 함수 내부의 코드를 실행하는 경우가 이에 해당합니다.
    간단한 예제 코드를 통해 실행 컨텍스트 환경을 시각적으로 보여드리도록 하겠습니다.
    주석에 주목하시기 바랍니다.

    ```javascript
    // 전역 실행 컨텍스트 환경

    if (true) {
        console.log('실행!');
    }

    eval(
        // eval 함수 실행 컨텍스트 환경

        '2 + 2',
    );

    function exec() {
        // 함수 실행 컨텍스트 환경

        console.log('실행!');
    }

    exec();
    ```

    ECMA Script에서는 실행 컨텍스트 환경의 생성을 *현재 실행되는 컨텍스트 환경에서 이 컨텍스트 환경과 관련 없는 실행 코드가 실행되면, 새로운 컨텍스트 환경이 생성되어 스택에 들어가고 그 제어권이 해당 컨텍스트 환경으로 이동한다*고 설명합니다.
    위의 예제 코드를 다시 살펴보겠습니다.

    ```javascript
    // 전역 실행 컨텍스트 환경

    if (true) {
        console.log('실행!');
    }

    eval(
        // 👈 전역 컨텍스트 환경과 상관없는 eval 함수 실행 코드가 실행됨! => eval 함수 컨텍스트 환경이 스택에 들어가고 제어권 역시 넘어감
        // eval 함수 실행 컨텍스트 환경

        '2 + 2',
    );

    function exec() {
        // 👈 전역 컨텍스트 환경과 상관없는 함수 실행 코드가 실행됨! => 함수 컨텍스트 환경이 스택에 들어가고 제어권 역시 넘어감
        // 함수 실행 컨텍스트 환경

        console.log('실행!');
    }

    exec();
    ```

    그렇다면 여기서 말하는 제어권이란 무엇일까요?
    다른 예제 코드를 통해 제어권에 대해 살펴보겠습니다.

    ```javascript
    // 전역 실행 컨텍스트 환경

    var a = 1;

    console.log(a); // 1

    function exec() {
        // 👈 전역 컨텍스트 환경과 상관없는 함수 실행 코드가 실행됨! => 함수 컨텍스트 환경이 스택에 들어가고 제어권 역시 넘어감
        // 함수 실행 컨텍스트 환경
        var a = 2;

        console.log(a);
    }

    exec();
    ```

    앞서 말씀드렸듯이 전역 컨텍스트 환경의 제어권이 함수 컨텍스트 환경으로 넘어가기 때문에 전역변수 a는 exec 함수 내에 a보다 우선적으로 적용되지 않습니다.
    물론, exec 함수 내에 a가 존재하지 않는다면 스코프 체인에 의해 전역변수 a가 출력됩니다.
    이는 이 파트의 범위를 넘어서는 것이므로 스코프 파트에서 다시 다루도록 하겠습니다.

2. 스택과 큐, 실행 컨텍스트

    본격적으로 실행 컨텍스트 환경을 살펴보기에 앞서 자료구조인 스택과 큐의 개념을 잠시 살펴보도록 하겠습니다.
    스택은 ~~출입구가 하나뿐인 깊은 우물 같은 데이터 구조~~라고 교재에서 설명하고 있으나, 개인적으로 저는 *쌓인 책 더미 📚*라는 표현을 좋아하고 또 직관적이라고 생각하기 때문에 책 더미에 비유하도록 하겠습니다.
    먼저 a라는 책이 존재합니다. 바닥에 놓습니다.
    그 다음 b라는 책이 존재합니다. a책 위에 놓습니다.
    그 다음 c라는 책이 존재합니다. b책 위에 놓습니다.
    마지막으로 d라는 책이 존재합니다. c책 위에 놓습니다.
    책 더미가 다음과 같이 쌓였습니다.

    d (위)  
    c  
    b  
    a (아래)

    이제 책을 읽습니다. 스택의 세계에서 ~~밑장 빼기~~는 허용되지 않으므로 a책을 읽기 위해서는 먼저 d를, 그 다음 c를, 그 다음 b를 읽어야만 합니다.
    개발자 사이에서 유명한 사이트인, '스택오버플로우'의 그 스택입니다.
    스택 오버 플로우를 설명하기 위해서는 저자의 비유인 *우물*이 좀 더 적합하긴 하겠습니다.
    책을 100권만 쌓을 수 있는 우물에 200권의 책을 쌓았다고 가정해 봅시다.
    당연히 책 더미가 우물 밖으로 우뚝 솟아나오는 것은 물론, 자칫하면 와르르 무너질 수도 있겠죠?
    많은 프로그래밍 언어들은 이처럼 스택이 넘칠 때 에러를 던집니다.
    ~~저도 재귀 호출이나, while문을 섣부르게 쓰다가 브라우저의 콜 스택을 폭파시키는 일이 빈번합니다(...)~~

    한편 큐는 ~~모두 열려있는 파이프~~를 떠올리면 된다고 교재에서 설명하고 있으나, 개인적으로 저는 *매표소 🎫*라는 표현을 좋아하고 또 직관적이라고 생각하기 때문에 매표소에 비유하도록 하겠습니다.
    물론 종류에 따라 양쪽 모두 입력과 출력이 가능한 큐도 있기 때문에 파이프가 좀 더 적합할 수 있으나 보통은 한쪽은 입력만, 다른쪽은 출력만을 담당하는 구조이기 때문에 매표소가 틀린 비유는 아닙니다.
    먼저 a라는 사람이 티켓을 사기 위해 줄을 섭니다. 매표소 창구 앞에 섭니다.
    그 다음 b라는 사람이 티켓을 사기 위해 줄을 섭니다. a사람 뒤에 섭니다.
    그 다음 c라는 사람이 티켓을 사기 위해 줄을 섭니다. b사람 뒤에 섭니다.
    그 다음 d라는 사람이 티켓을 사기 위해 줄을 섭니다. d사람 뒤에 섭니다.  
    다음과 같이 줄을 섰습니다.

    (매표소) a b c d

    이제 차례대로 티켓을 구매합니다. 큐의 세계에서 ~~새치기~~는 허용되지 않으므로 d사람이 티켓을 사기 위해서는 먼저 a사람이, 그 다음 b사람이, 그 다음 c사람이 티켓을 사야합니다.

    앞서 실행 컨텍스트를 *실행할 코드에 제공할 환경 정보들을 모아놓은 객체*라고 했습니다.
    동일한 환경에 있는 코드들을, 실행할 때 필요한 환경 정보들을 모아 컨텍스트를 구성(실행 컨텍스트 객체)하고 이를 콜 스택에 쌓아 올렸다가, 가장 위에 쌓여 있는 컨텍스트와 관련 있는 코드들을 실행하는 식으로
    전체 코드의 환경과 순서를 보장합니다.
    여기서 말하는 '동일한 환경'이란 위의 예제의 주석으로 설명한 **실행 컨텍스트 환경**을 말합니다.
    전역 실행 컨텍스트 환경이나, eval 함수 실행 컨텍스트 환경을 제외하면 우리가 흔히 실행 컨텍스트를 구성하는 방법은 *함수를 실행*하는 것뿐입니다.
    중요한 것은 *선언*이 아니고 *실행*입니다.
    위의 예제 코드의 주석에서 마치 함수가 선언될 때 실행 컨텍스트 환경이 구성되는 것처럼 오해할 수 있으나, 이는 시각적으로 쉽게 보여주기 위함이며, 컨텍스트 환경은 함수가 *선언*될 때가 아니라 *실행*될 때 구성됩니다.
    ES6에서는 블록에 의해서도 새로운 실행 컨텍스트 환경이 생성됩니다.
    const와 let 키워드로 선언한 변수가 이에 해당합니다.
    예제 코드를 통해 살펴보겠습니다.

    ```javascript
    // 전역 실행 컨텍스트 환경
    const a = 1;

    console.log(a); // 1

    function exec() {
        // 함수 실행 컨텍스트 환경
        const a = 2;

        console.log(a);

        if (true) {
            // if문 실행 컨텍스트 환경
            const a = 3;

            console.log(a);
        }
    }

    exec(); // 2 / 3
    ```

    만일 exec 함수 내의 if문 블록이 함수 실행 컨텍스트 환경의 제어권에 있었다면 if문 내에 const 키워드는 재선언이나 재할당할 수 없으므로 이미 선언돼 있다는 에러가 발생했을 것입니다.
    하지만 const와 let 키워드는 블록 실행 컨텍스트를 따르므로 위의 예제 코드는 아무런 에러가 발생하지 않습니다.

3. 실행 컨텍스트 생성 과정

    ```javascript
    // ---------------------------------- (1)
    var a = 1;

    function outer() {
        function inner() {
            console.log(a); // undefined

            var a = 3;
        }

        inner(); // --------------------- (2)
        console.log(a); // 1
    }

    outer(); // ------------------------- (3)
    console.log(a);
    ```

    실행 컨텍스트는 환경은 선언이 아니라 *실행*될 때 구성된다고 앞서 말씀드렸습니다.
    위의 예제 코드와 같이 자바스크립트 코드를 실행하는 순간(1) 전역 컨텍스트 환경이 콜 스택에 담깁니다.
    전역 컨텍스트 환경이라는 개념은 일반적인 실행 컨텍스트 환경과 특별히 다를 것이 없습니다.
    굳이 차이점을 찾자면 전역 컨텍스트 환경이 관여하는 대상은 함수가 아닌 전역 공간이기 때문에 arguments가 없습니다.
    전역 공간을 둘러싼 외부 스코프란 존재할 수 없기 때문에 스코프체인 상에는 전역 스코프 하나만 존재합니다.
    이런 성질들은 구조상 당연히 그럴 수밖에 없는 것입니다.

    | 콜스택        |
    | ------------- |
    |               |
    |               |
    |               |
    | 전역 컨텍스트 |

    (2)에서 inner 함수의 실행 컨텍스트 환경이 콜스택 가장 위에 담기면 outer 컨텍스트 환경과 관련된 코드의 실행을 중단하고 inner 함수 내부의 코드를 순서대로 진행합니다.
    다음과 같이 진행됩니다.

    | 콜스택        |
    | ------------- |
    |               |
    |               |
    | outer         |
    | 전역 컨텍스트 |

    | 콜스택        |
    | ------------- |
    |               |
    | inner         |
    | outer         |
    | 전역 컨텍스트 |

    | 콜스택        |
    | ------------- |
    |               |
    |               |
    | outer         |
    | 전역 컨텍스트 |

    | 콜스택        |
    | ------------- |
    |               |
    |               |
    |               |
    | 전역 컨텍스트 |

    | 콜스택 |
    | ------ |
    |        |
    |        |
    |        |
    |        |

4. 실행 컨텍스트 과정

    3번에서는 실행 컨텍스트 환경이 *생성*되어 콜 스택에서 처리되는 과정에 대해 살펴봤습니다.
    4번에서는 실행 컨텍스트 내부에서 일어나는 일과, 코드가 어떻게 실행되는지, 즉 실행 컨텍스트 과정에 대해 살펴보겠습니다.
    쉽게 설명하면, 3번에서는 실행 컨텍스트 환경이라는 컨테이너가 어떻게 콜 스택에 적재되고 제거되는지 살펴봤다면, 4번에서는 각각의 컨테이너 안에서 일어나는 과정에 대한 설명입니다.
    실행 컨텍스트 환경이 활성화될 때 자바스크립트 엔진은 해당 컨텍스트 환경 내부의 코드들을(책에서는 관련된이라고 서술하고 있으나 모호한 표현이라 이렇게 설명합니다.) 실행하는데 필요한 *환경 정보*들을 수집해서 **실행 컨텍스트 객체**에 저장합니다.
    이 객체는 자바스크립트 엔진이 활용할 목적으로 생성될 뿐 개발자가 코드를 통해 확인할 수는 없습니다.
    여기에 담기는 정보들은 다음과 같습니다.

    - VariableEnvironment(변수 환경 객체)
    - LexicalEnvironment(어휘 환경 객체)
    - ThisBinding(this 바인딩 객체)

    쉽게 설명하면 실행 컨텍스트 환경이라는 컨테이너 내부에 VariableEnvironment라는 컨테이너와, LexicalEnvironment라는 컨테이너, ThisBinding이라는 컨테이너가 있다고 생각하면 됩니다.
    먼저 LexicalEnvironment 객체를 살펴 보겠습니다.

    ```javascript
    LexicalEnvironment = {
        EnviromentRecord: {},
        OutterEnviromentReference: {},
    };
    ```

    LexicalEnvironment 객체의 내부는 위와 같습니다.
    현재 실행 컨텍스트 환경 내부의 함수 혹은 변수가 저장되는 Environment Record라는 공간과, 외부 컨텍스트 환경을 *참조*하는 Outter Enviroment Reference 공간입니다.
    외부 컨텍스트 환경을 *참조*하기 때문에 record가 아니라 reference라는 이름이 붙었습니다.
    environment record 객체 내부는 다음과 같이 Object 안에 실제 값들이 저장되는 Environment Record라는 공간과,
    with문과 같이 식별자를 어떤 특정 객체 A의 속성으로 취급할 때 A를 바인딩 해주는 Object Environment Record라는 공간으로 구성돼 있습니다.

    ```javascript
    LexicalEnvironment = {
        EnviromentRecord: {
            DeclartionEnvironmentRecord = {
                a: 33,
                b: 'Hello World',
            },
            ObjectEnvironmentRecord = {
                bindObject: [object],
            }
        },
        OutterEnviromentReference: {}
    }
    ```

    잠시 with문을 살펴보도록 하겠습니다.

    ```javascript
    const obj = {
        name: 'soulcactus',
    };

    with (obj) {
        name = 'hwizzzang';
    }

    console.log(obj['name']); // hwizzzang
    ```

    해당 with문을 실행시 ObjectEnvironmentRecord에 바인딩되는 객체는 obj입니다.
    DeclartionEnvironmentRecord에 담기는 정보는 아래 예제 코드를 통해 살펴보겠습니다.

    ```javascript
    function sum(x, y) {
        var result = x + y;
        var etc = function() {
            console.log('good');
        };

        function msg() {
            return result;
        }
    }

    sum(10, 20);
    ```

    위와 같은 코드가 있다고 가정합니다. 자바스크립트 엔진이 sum(10, 20)을 만나면 함수 sum에 대한 실행 컨텍스트 환경을 생성합니다.
    그리고 우선적으로 this를 찾아서 thisBinding 객체에 바인딩합니다.
    벌써 컨테이너 세 개의 컨테이너 중 하나인 thisBinding의 정체가 해결됐습니다. ✌
    sum은 일반 함수이므로 이 경우 thisBinding에 바인딩된 객체는 window 객체입니다.
    함수가 실행되면 다음과 같이 EnviromentRecord 객체에 함수의 매개변수인 x와 y값이 저장됩니다.

    ```javascript
    LexicalEnvironment = {
        EnviromentRecord: {
            DeclartionEnvironmentRecord = {
                x: 10,
                y: 20
            },
            /* ... */
        },
        OutterEnviromentReference: global
    }
    ```

    그 다음으로는 함수 내부에 선업된 것들을 가져오고, 함수 내부에서 사용할 수 있는 arguments를 세팅합니다.
    그러고 나서 선언된 변수들을 가져오는데, 할당 값을 바로 저장하지는 않고 undefined를 저장합니다.

    ```javascript
    LexicalEnvironment = {
        EnviromentRecord: {
            DeclartionEnvironmentRecord = {
                x: 10,
                y: 20,
                arguments: Arguments Object,
                result: undefined,
                etc: undefined,
            },
            /* ... */
        },
        OutterEnviromentReference: global
    }
    ```

    우리는 이미 호이스팅이 변수 *할당*이 아닌 *선언*만을 위로 끌어올린다는 사실을 알고 있습니다.
    그 사실을 기억하고 위의 DeclartionEnvironmentRecord에 담긴 변수 정보를 다시 살펴보시기 바랍니다.
    result라는 이름을 가진 정보는 저장됐지만, 그것이 가리키는 값은 아직 저장되지 않았습니다.
    사실, 이것이 바로 호이스팅의 실체입니다. 뒷 부분에서 더 자세히 알아보겠지만, ~~정말 이게 전부입니다.~~ 😉
    자, 이제 함수가 실행되면 LexicalEnvironment의 내부에 어떤 일이 일어나는 지 살펴보도록 하겠습니다.

    var result = x + y를 만나면 계산 후에 result에 저장하고, var etc = function() { console.log('good') }을 만나면 해당 익명 함수의 참조를 etc에 저장합니다.

    ```javascript
    LexicalEnvironment = {
        EnviromentRecord: {
            DeclartionEnvironmentRecord = {
                x: 10,
                y: 20,
                arguments: Arguments Object,
                result: 30,
                etc: Function Reference,
            },
            /* ... */
        },
        OutterEnviromentReference: global
    }
    ```

    함수 sum의 상위 컨텍스트 환경은 전역 환경이므로 OutterEnviromentReference는 global이 되고 상위 컨텍스트 환경에 있는 값을 사용하고자 할 때는 OutterEnviromentReference를 참조합니다.
    사실, 이것이 바로 스코프체인의 실체입니다. 다음 파트에서 더 자세히 알아보겠지만, ~~정말 이게 전부입니다.~~ 😉

    그렇다면 VariableEnvironment 객체는 무엇일까요?
    기본적으로는 지금까지 살펴본 LexicalEnvironment 객체와 동일합니다. 사실 지금까지 살펴본 LexicalEnvrionment 객체의 생성 과정은 VariableEnvironment 객체의 생성 과정이었습니다.
    ~~갑자기 무슨 말이냐고요?~~ 순서 상 LexicalEnvrionment 객체보다 VariableEnvrionment 객체가 먼저 생성되기 때문입니다. 그렇다면 두 객체는 같을까요?
    _거의_ 같다고 할 수 있습니다만, VariableEnvironment 객체는 앞에 with문과 함께 언급한 ObjectEnvironmentRecord 객체를 가지지 않습니다.
    동적으로 변경되는 사항을 반영하지 않습니다. 다음 예제 코드를 통해 살펴보겠습니다.

    ```javascript
    var a = 10;

    function foo() {
        console.log(a);
    }

    with ({ a: 20 }) {
        var bar = function() {
            console.log(a);
        };

        foo(); // 10, from VariableEnvrionment
        bar(); // 20,  from LexicalEnvrionment
    }

    foo(); // 10
    bar(); // still 20
    ```

    with문을 이용한 새로운 a값인 20은 VariableEnvrionment 객체에는 저장되지 않는 값입니다.
    이처럼 LexicalEnvrionment 객체와 달리, VariableEnvrionment 객체는 변경 사항이 반영되지 않습니다.
    ~~with문은 eval과 마찬가지로 대표적인 암흑의 구문이므로~~ 사실 with문 등을 사용하지 않는 이상 두 객체는 동일하다고 생각하시면 됩니다.
    정리하면, VariableEnvrionment 객체에 담기는 내용은 LexicalEnvrionment 객체와 같지만 최초 실행시의 스냅샷을 유지한다는 점이 다릅니다.
    실행 컨텍스트 객체를 생성할 때, VariableEnvrionment 객체에 정보를 먼저 담은 다음, 이를 그대로 복사해서 LexicalEnvrionment 객체를 만들고, 이후에는 LexicalEnvrionment 객체를 주로 활용합니다.
    이로써 세 가지 컨테이너의 정체가 모두 해결됐습니다.
