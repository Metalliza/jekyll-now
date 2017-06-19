[Variable Declarations](https://www.typescriptlang.org/docs/handbook/variable-declarations.html)

## var
```javascript
var a = 10;
```
>전통적 JavaScript에서 변수 선언하는 방법.
```javascript
function f() {
    var message = "Hello, world!";
    return message;
}
```
```javascript
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // returns '11'
```
>위의 예제에서 함수 **g**는 함수 **f**에서 선언된 변수**a**를 사용합니다. 함수 **g**가 호출 될 때마다 변수 **a**의 값은 함수 **f**의 변수 **a** 값을 사용합니다. 그리고 함수 **f**가 실행 되고 함수 **g**를 호출하여 변수 **a**에 액세스하여 값을 수정할 수도 있습니다.
```javascript
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns '2'
```
## Scope rules
```javascript
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```
```javascript
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```
>**for**-loop가 변수 **i**를 덮어씀
## Variable capturing quirks
```javascript
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```
>**setTimeout**은 몇 밀리초 후에 함수를 실행하지만, **for** loop가 실행을 멈춘 후에 실행됩니다. **for** loop가 실행을 마치면 **i**의 값은 **10**입니다. 따라서 주어진 함수가 호출 될 때마다 **10**이 출력됩니다!

해결법은 IIFE - Immediately Invoked Function Expression
```javascript
for (var i = 0; i < 10; i++) {
    // 'i'의 현재 상태를 캡처
    // 현재 값으로 함수를 호출
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```
## let
```javascript
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```
>코드에서 setTimeout에 전달하는 함수 표현식은 실제로 동일 Scope에서 같은 i를 참조합니다. 그게 무슨 뜻인지 잠시 생각해 봅시다. setTimeout은 몇 밀리초 후에 함수를 실행하지만, for loop가 실행을 멈춘 후에 실행됩니다. for loop가 실행을 마치면 i의 값은 10입니다. 따라서 주어진 함수가 호출 될 때마다 10이 출력됩니다!

```javascript
for (var i = 0; i < 10; i++) {
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```
>일반적인 해결 방법은 각 반복마다 i를 캡처하는 IIFE(Immediately Invoked Function Expression : 함수를 바로 호출 하는 표현식)를 사용하는 것입니다.
```javascript
for (let i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```
>let 선언문은 루프의 일부로 선언 될 때 크게 다른 행동을 합니다. 루프 자체에 새로운 환경을 도입하기보다는 이러한 선언은 반복마다 새로운 Scope을 만듭니다. 어쨌든 IIFE를 사용하여이 작업을 수행했던 이전의 setTimeout 예제를 let 선언을 이용하여 변경할 수 있습니다.

## 재선언 과 Shadow
>var 선언자를 이용한 변수선언은 선언 횟수가 중요하지 않습니다.
```javascript
function f(x) {
    var x;
    var x;
    if (true) {
        var x;
    }
}
```
>위의 예제에서 x의 모든 선언은 실제로 같은 x를 참조하며 이것은 완벽하게 유효합니다. 이런 방식의 코딩은 종종 버그의 원인이되곤 합니다. 하지만 let 선언자를 이용한 변수 선언은 이러한 방식을 허용하지 않습니다.
```javascript
let x = 10;
let x = 20; // error: 같은 Scope 내에서 변수 'x'를 재선언 할 수 없습니다.
```
>아래 예제와 같이 Block-scope 변수가 아니어도 변수의 재선언시 Typescript에서 문제가 있음을 알려줍니다.
```javascript
function f(x) {
    let x = 100; // error: parameter 변수 선언에 간섭하고 있습니다.
}
function g() {
    let x = 100;
    var x = 100; // error: 변수 'x'를 두개 선언할 수 없습니다.
}
```
>하지만 Block-scope 변수가 Function-scope에 절대로 선언 될 수 없다는 말은 아닙니다. 블록 범위 변수는 뚜렷하게 다른 블록 내에서 선언되어야만 합니다.
```javascript
function f(condition, x) {
    if (condition) {
        let x = 100;
        return x;
    }
    return x;
}
f(false, 0); // returns '0'
f(true, 0);  // returns '100'
```
>중첩 된 Scope에 기존의 변수 이름을 사용하는 것을 Shadow라고합니다. 하지만 우발적으로 Shadow를 사용할 경우 버그를 유발할 수 있기 때문에 양날의 검일수 있습니다. 아래의 예제는 let 변수를 사용하여 이전에 작성했던 sumMatrix 함수를 다시 작성했습니다.
```javascript
function sumMatrix(matrix: number[][]) {
    let sum = 0;
    for (let i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (let i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }
    return sum;
}
```
>이 버전의 for loop는 실제로 내부 for loop의 i가 외부 for loop의 i를 Shadow 하기 때문에 실제로 합계를 올바르게 수행합니다.
>일부 코드에서 Shadow가 필요할 수도 있지만 일반적으로 더 명확한 코드를 작성하기 위해 Shadow는 피해야합니다. 

## const
다시 할당은 안되나 참조값은 변경 가능
```javascript
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

**let vs. const**

>비슷한 “Block-scope” 유형을 가진 두 가지 선언이 있다고 가정 할 때, let과 const중 어떤 것을 사용할지 스스로 선택해야 합니다. 간단히 힌트를 드리자면 다음과 같습니다.

>최소 권한의 원칙(Principle of least privilege)을 적용하면 수정하려는 모든 선언은 const를 사용해야 합니다. 변수에 write 할 필요가 없는 경우, 같은 코드를 이용하여 협업하는 사람들이 자동으로 객체에 값을 쓰지 않아야 하는 경우, 변수에 실제로 재 할당이 필요하는지 등의 여부를 고려해야합니다. const를 사용하면 데이터 흐름에 대해 추론 할 때 코드를 더 예측 가능하게 만듭니다.

>많은 사용자들은 간결함을 선호 할 것 이기 때문에 var는 더이상 사용하지 않고 let을 사용할 것입니다. 이 핸드북의 대부분은 let 선언문을 사용합니다.

## Destructuring
[MDN](https://developer.mozilla.org/ko-KR/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

```javascript
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```
```javascript
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```
```javascript
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
let [, second, , fourth] = [1, 2, 3, 4];
```
### Object destructuring
```javascript
let o = {
    a: "foo",
    b: 12,
    c: "bar"
};
let { a, b } = o;
```
>선언 없이 할당
```javascript
({ a, b } = { a: "baz", b: 101 });
```
**()** 주목
```javascript
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;
```
### Property 재명명(renaming)
```javascript
let { a: newName1, b: newName2 } = o;
```
```javascript
let newName1 = o.a;
let newName2 = o.b;
```
```javascript
console.log(o.a, o.b);
console.log(newName1, newName2);
```

>혼란스럽지만 여기 콜론은 타입을 나타내지 않습니다. 타입을 지정하는 경우 전체 destructuring 된 후에 타입을 지정해야 합니다.
```javascript
let { a, b }: { a: string, b: number } = o;
```
`foo 12`

### 기본값 지정
```javascript
function keepWholeObject(wholeObject: { a: string, b?: number }) {
    let { a, b = 1001 } = wholeObject;
}
```

### Function 선언
```javascript
type C = { a: string, b?: number }
function f({ a, b }: C): void {
    // ...
}
```
>Parameter의 기본값을 지정하는 것이 더 일반적이며, Destructuring시 정확한 기본값을 가져 오는 것은 까다로울 수 있습니다. 우선, 기본값 앞에 타입을 적어야 하는 것을 잊지 말아야 합니다.
```javascript
function f({ a, b } = { a: "", b: 0 }): void {
    // ...
}
f(); // ok, default to { a: "", b: 0 }
```

```javascript
function f({ a, b = 0 } = { a: "" }): void {
    // ...
}
f({ a: "yes" }); // ok, default b = 0
f(); // ok, default to { a: "" }, which then defaults b = 0
f({}); // error, 'a' is required if you supply an argument
```
>조심해서 Destructuring을 사용하십시오. 앞의 예제에서 보여 주듯이 단순한 Destructuring 표현을 제외하고는 혼란스러울 수 있습니다. 특히 이름 바꾸기, 기본값, type annotation을 사용하지 않아도 이해하기 힘든 깊이 중첩된 Destructuring에서는 특히 그렇습니다. Destructuring 표현은 단순하면서 최소한 유지하십시오. 언제든지 여러분이 생성한 Desctructuring을 할당해서 사용할 수 있습니다.

## Spread
>Spread 연산자는 Destructuring의 반대입니다. 배열을 다른 배열로 펼치거나(Spread) 객체를 다른 객체로 퍼뜨릴(Spread) 수 있습니다.

```javascript
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
```
`[0, 1, 2, 3, 4, 5]`
>객체 Spread
```javascript
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };

console.log(search);
```
`{ defaults: { food: 'spicy', price: '$$', ambiance: 'noisy' },
  food: 'rich' }`

>객체 Spread에는 몇 가지 다른 놀라운 limit
1. 자신의 Enumerable property(열거 가능한 속성)만 포함됩니다. 이는 객체의 인스턴스를 Spread 할 때 메서드가 손실된다는 것을 의미합니다
2. Typescript 컴파일러는 generic function의 Parameter 변수의 스프레드를 허용하지 않습니다. 이 기능은 향후 버전에서 지원될 것으로 예상됩니다.
```javascript
class C {
  p = 12;
  m() {
  }
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```
