# 실행 컨텍스트와 호이스팅

## 🧐 실행 컨텍스트

   ### 1. 실행 컨텍스트란 무엇인가요?

   ECMAScript에서는 실행 컨텍스트를 **"실행가능한 코드를 형상화하고 구분하는 추상적인 개념"** 으로 기술하고 있으며, 이를 다시 정의한다면  **"실행 가능한 코드가 실행되기 위해 필요한 환경"** 이라고 할 수 있습니다.

   여기서 말하는 실행 가능한 코드에는 **전역 코드**, **eval 코드**, **함수 코드**가 있습니다.

   ---
   * 전역 코드 : 전역 영역에 존재하는 코드
   * eval 코드 : eval() 함수로 실행되는 코드
   * 함수 코드 : 함수 내에 존재하는 코드
   ---

   자동으로 생성되는 전역 영역에 존재하는 코드, 악마로 취급받는 eval()을 제외하면 우리가 흔히 실행 컨텍스트를 수정하는 방법은 함수를 실행하는 것뿐입니다.

   실행가능한 코드들을 실행할 때 필요한 환경 정보들을 모아 컨텍스트를 구성하며, 이를 **콜 스택에 쌓아 올렸다가 (push), 가장 위에 쌓여있는 컨텍스트와 관련있는 코드들을 실행하는 식으로 전체 코드의 환경과 순서를 보장**합니다

   ---
   * 콜 스택 : **함수의 호출을 기록하는 자료구조**로, 프로그램에서 우리가 어디에 있는지를 기본적으로 기록하는 데이터 구조입니다. 어떤 함수를 실행할 때 해당 함수의 기록을 스택 맨 위에 추가(push)하고, 함수가 결과값을 반환하면 스택에 쌓여있던 함수는 제거(pop)됩니다.

   * 스택 : 나중에 집어넣은 데이터가 먼저 나옵니다. 이 특징을 **LIFO**(First In First Out)라고 합니다. 데이터를 집어넣는 push, 데이터를 추출하는 pop, 맨 나중에 집어넣은 데이터를 확인하는 peek 등의 작업을 할 수 있습니다.

   * 큐 : 먼저 집어넣은 데이터가 먼저 나옵니다. 이 특징을 **FIFO**(First In First Out)라고 합니다. 데이터를 집어넣는 enqueue, 데이터를 추출하는 dequeue 등의 작업을 할 수 있습니다.

   * 덱 : **자료의 입력과 출력을 양 쪽에서 가능**하게 하여 스택과 큐에 비해 자유도가 높은 자료구조입니다.
   ---

   ### 2. 실행 컨텍스트와 콜 스택

   여기서는 콜 스택에 실행 컨텍스트가 어떤 순서로 쌓이고, 어떤 순서로 코드 실행에 관여하는 지만 확인할 수 있으면 됩니다. 밑 코드에 대한 예제는 뒤에서 더 자세히 알아보도록 하겠습니다.

   ```javascript

      // --------------------- (1)

      var a = 1;

      function outer() {
         function inner() {
            console.log(a);
            var a = 3;
         }
         inner(); // --------- (2)
         console.log(1)
      }

      outer(); // ------------ (3)
      console.log(a);

   ```
   <!-- 코드에 대한 설명 추가 -->
   <!-- 코드에 대한 설명 추가 -->
   <!-- 코드에 대한 설명 추가 -->

   스택 구조를 생각해보았을 때, 기존의 컨텍스트는 새로 쌓인 컨텍스트보다 아래에 위치할 수밖에 없기 때문에 한 실행 컨텍스트가 콜 스택의 맨 위에 쌓이는 순간이 곧 현재 실행할 코드에 관여하게 되는 시점임을 알 수 있습니다.

   어떤 실행 컨텍스트가 활성화될 때 자바스크립트 엔진은 해당 컨텍스트에 관련된 코드들을 실행하는 데 필요한 환경 정보들을 수집하여 실행 컨텍스트 객체에 저장하는데요.

   여기에 담기는 정보에는 **VariableEnvironment**, **LexicalEnvironment**, **thisBinding** 이 있습니다.

   ---
   * VariableEnvironment : 현재 컨텍스트 내의 식별자들에 대한 정보 + 외부환경 정보. 선언 시점의 LexicalEnvironment의 스냅샷으로, 함수 실행 도중에 일어난 변경 사항은 반영되지 않고 초기 상태를 유지함.
   * LexicalEnvironment : 처음에는 VariableEnvironment와 같지만 함수 실행 도중에 일어난 변경 사항이 실시간으로 반영됨.
   * thisBinding : 식별자가 바라봐야 할 대상 객체.
   ---

   위의 친구들은 다음 내용에서 뜯어보겠습니다.

## 🧐 VariableEnvironment

   VariableEnvironment에 담기는 내용은 LexicalEnvironment와 같지만 **최초 실행 시의 스냅샷을 유지**한다는 점이 다르며, 변경 사항은 반영되지 않습니다.

   생성 후에 내부 값이 변할 수 있는 LexicalEnvironment Object와는 달리 VariableEnvironment Object는 내부에서 선언된 변수(Variables), 함수 선언(Function Declarations), 함수 매개 변수(Formal Parameters) 들을 저장하기 때문에 Hoisting 등 this 키워드를 이용한 Expression에 의해 새로운 변수/ 함수가 등장하더라도 절대로 값이 변하지 않습니다.

   실행 컨텍스트를 생성할 때 먼저 VariableEnvironment에 정보를 담은 후, 이를 그대로 복사하여 LexicalEnvironment를 만들고, 이후 LexicalEnvironment를 주로 활용하게 됩니다.

## 🧐 LexicalEnvironment

   실행 컨텍스트 생성 초기 시점에는 VariableEnvironment와 정확히 같은 값을 가지나, 컨텍스트 내부에서 Scope Augmentation(스코프체인 확정)이 실행될 시에는 VariableEnvironment와는 달리 **새로운 Identifier와 그의 Reference 값이 추가**됩니다.

   ### 1. environmentRecord

   현재 컨텍스트와 관련된 코드의 **식별자 정보**들이 저장됩니다.
   저장되는 식별자 정보에는 함수의 매개변수 식별자, 함수 자체, var로 선언된 변수의 식별자가 있습니다.

   ### 2. Hoisting(호이스팅)

   environmentRecord는 코드 실행 전 컨텍스트 내부를 처음부터 끝까지 훑으며 순서대로 수집합니다. 

   각 정보들을 수집하는 과정을 마쳤더라도 아직 실행 컨텍스트가 관여할 코드들은 실행되기 전의 상태인데, 이러한 상태임에도 불구하고 자바스크립트 엔진은 이미 실행할 컨텍스트 내부에 있는 모든 식별자들을 알고 있게 됩니다.
   그렇기에 '자바스크립트 엔진은 식별자들을 최상단으로 끌어올려놓은 다음 실제 코드를 실행한다'라고 생각해도 코드를 해석하는 데 문제될 것이 없을 것입니다.

   이 부분에서, 자바스크립트 엔진이 실제로 식별자들을 최상단으로 끌어올리지는 않지만 그렇게 간주하자!는 **호이스팅**이 등장합니다. (두둥)

   ```javascript

   function a(x) {
      console.log(x); // (1)
      var x;
      console.log(x); // (2)
      var x = 2;
      console.log(x); // (3)
   }

   a(1);

   ```

   로그가 어떻게 찍힐까요? 호이스팅이 안 되었을 경우, 되었을 경우로 나눠 결과를 예상해보겠습니다.

   * 호이스팅이 안 되었을 경우 : (1)에는 함수 호출 시 1이 출력, (2)는 선언된 변수 x에 할당한 값이 없으므로 undefined가 출력, (3)은 2가 출력될 것이라 생각할 수 있습니다. 실제로 코드 실행 시 어떤 결과가 나올까요? 생각과 같을까요? *흠*

   * 호이스팅이 되었을 경우 : 