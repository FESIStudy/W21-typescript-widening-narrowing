# 타입스크립트란?
정적 타입검사와 객체 지향 프로그래밍 지원을 제공하는 프로그래밍 언어이며 `.ts` 확장자를 사용합니다.

### 주요 특징
- **정적 타입 시스템** 제공 (컴파일 타임에 오류 검출 가능)
- 기존 **JavaScript 문법 100% 호환**
- ES6 이상의 최신 문법을 선행 지원
- 인터페이스, 제네릭, 열거형, 유니언 타입 등 다양한 타입 시스템 도입
- 개발 도구와 IDE 지원이 뛰어남

### 목적
- 대규모 프로젝트에서 코드의 안정성, 가독성, 유지보수성 향상
- 런타임 오류를 줄이고, 개발 초기 단계에서 문제를 발견할 수 있게 함

## 타입 시스템의 역할
타입 시스템은 값의 타입을 추적하고 관리함으로 다음과 같은 역할을 수행합니다

### 1. 오류 방지
- 코드 작성 시점에 타입 오류를 감지하여 런타임 오류 감소
```ts
function add(a: number, b: number): number {
  return a + b;
}
add(1,'2'); //컴파일 에러
```
### 2. 코드의 명확성과 문서화
- 함수나 객체의 인터페이스에 타입을 지정함으로써 의도가 명확해짐
- 협업에서 커뮤니케이션이 쉬움
### 3. 개발 도구 지원 강화
- 자동 완성, 타입추론, 오류 표시, 리팩토링 기능 향상
### 4. 리팩토링 안정성
- 타입 정보를 바탕으로 코드를 안전하게 변경할 수 있음

## 타입 추론(Type Inference)의 과정과 한계
### 🔷 정의
타입스크립트는 **명시적 타입 선언이 없더라도**, 코드의 구조를 분석해 **자동으로 타입을 추론**합니다.
```ts
let x = 5; //number로 타입 추론
```
### 🔷 과정
타입스크립트는 다음을 통해 타입을 추론합니다:
1. 초기값에 의한 추론 (Variable Initialization)
1. 리턴 값에 의한 추론 (Function Return Type)
1. 컨텍스트 기반 추론 (Contextual Typing)
```ts
window.onmousedown = function (event) {
  console.log(event.button); // event 타입을 MouseEvent로 추론
};
```
### 🔷 한계
1. 복잡한 상황에서는 정확한 추론이 어려움
1. 개발자가 의도한 타입과 추론된 타입이 다를 수 있음
1. any로 추론되는 경우가 있음 (ex. 빈 배열 등)
```ts
let arr = []; // any[]
arr.push(1);  // number도 넣을 수 있고
arr.push('a'); // string도 넣을 수 있음 ⇒ 타입 안정성 저하
```
이런 문제 때문에 **명시적 타입 지정**이 중요합니다.

## Strict 모드
TypeScript의 최고 수준의 타입 안정성 검사를 활성화하는 모드입니다. `tsconfig.json`에서 `strict: true`로 설정할 수 있습니다.
```json
{
  "compilerOptions": {
    "strict": true
  }
}
```
이 옵션을 켜면 noImplicitAny, strictNullChecks, strictFunctionTypes, strictBindCallApply, strictPropertyInitialization, alwaysStrict 등이 모두 자동으로 활성화됩니다.

### 🔷 타입 추론에 미치는 영향
- 더 엄격한 추론을 수행
- 암묵적 any 타입이 허용되지 않음
- null과 undefined에 대한 엄격한 구분이 적용됨
- 미초기화 변수, 필드에 대해 경고 발생

# 타입 넓히기 Widening (타입 확장)
타입스크립트가 리터럴 타입을 더 일반적인 타입으로 자동 변환하는 것
```ts
let a = 'hello'; // a: string (리터럴 'hello' → string으로 확장됨)
```
하지만 `as const`나 `const`로 선언하면 widening이 일어나지 않음:
```ts
export const MESSAGE_STATUS = {
  ID: 'ID',
  SYSTEM_CONTEXT: 'SYSTEM_CONTEXT',
  ANALYZING_SEMANTIC: 'ANALYZING_SEMANTIC',
  END: 'END',
  ERROR: 'ERROR',
} as const;

let messageInfo = MESSAGE_STATUS.ID

messageInfo='123'; //에러가 발생함
```

## Widening 발생하는 상황
- 타입스크립트는 값을 변수에 할당할 때 해당 값을 기반으로 타입을 추론합니다. 이때 변수의 선언 방식(let, const, 타입 명시 여부 등)에 따라 리터럴 타입이 더 넓은 타입으로 추론(widen)될 수 있습니다.

```ts
let str = "hello";
```

- `str`의 타입은 `"hello"`가 아니라 `string`입니다.
- 리터럴 타입이 `string`이라는  이것이 **widening**입니다.

widening이 발생하는 대표적인 상황:
| 상황        | 설명                                 | 예시                                              |
| --------- | ---------------------------------- | ----------------------------------------------- |
| `let` 선언  | `let`은 재할당 가능 → 타입스크립트가 일반 타입으로 추론 | `let num = 1; // type: number`                  |
| 타입 명시 없음  | 타입을 명시하지 않으면 widening 가능성이 높음      | `let flag = true; // type: boolean`             |
| 객체 속성 초기화 | 객체 속성도 widening 될 수 있음             | `let obj = { status: "ok" }; // status: string` |

## Widened Literal Types
 리터럴 타입은 `"hello"`,`42`,`true` 같이 값 그 자체를 타입으로 가지는 타입입니다.
하지만 다음과 같은 경우엔 wideing이 발생하여 widened literal type, 즉 일반 타입으로 바뀝니다.

```ts
let a = "apple";   //type: string
const b = "apple"; //type: "apple"
```

| 선언 방식               | 추론 타입     | 이유                            |
| ------------------- | --------- | ----------------------------- |
| `let a = "apple"`   | `string`  | 리터럴 `"apple"`이 일반 문자열로 확장됨    |
| `const b = "apple"` | `"apple"` | 재할당 불가 → 타입 좁혀짐 (no widening) |

객체의 속성도 마찬가지:
```ts
let fruit = { name: "apple" };
```
즉, `name`의 타입은 `"apple"`이 아닌 `string`으로 widening됨.

## as const로 Widening 방지하기
타입스크립트는 **as const**를 사용하면 변수나 객체의 모든 속성을 리터럴 타입 + readonly로 고정할 수 있습니다.
### 문자열 상수 고정
```ts
let x = "hello";       // x: string
let y = "hello" as const; // y: "hello"
```
### 객체 리터럴 고정
```ts
let obj1 = {
  kind: "circle",
  radius: 10
};
// obj1.kind: string

let obj2 = {
  kind: "circle",
  radius: 10
} as const;
// obj2.kind: "circle", obj2.radius: 10 (리터럴 타입)
// obj2 전체가 readonly로 처리됨
```
`as const`의 효과 정리:
| 효과                | 결과                                         |
| ----------------- | ------------------------------------------ |
| 리터럴 타입 유지         | `"apple"` 같은 타입이 `string`으로 widening 되지 않음 |
| 객체 속성 readonly 처리 | `readonly` 속성으로 고정되어 변경 불가                 |
| 튜플 고정 가능          | `[1, 2] as const` → `readonly [1, 2]`      |

## Zustand store 에서 widening 제어
Zustand는 보통 다음처럼 `create` 함수로 스토어를 생성합니다:
### ❌ widening된 경우 (문제 발생 가능)
```ts
const useStore = create(() => ({
  status: "idle", // ← string으로 widening
}));
```
- 위 코드에서 `status`는 `"idle"`이 아니라 `string`으로 추론됨
- 따라서 `"idle" | "loading" | "success"` 등 유니언 타입으로 비교할 때 타입 오류가 나지 않거나, **정확한 타입 제한이 작동하지 않음.**

### ✅ widening 방지 (`as const` 활용)
```ts
const useStore = create(() => ({
  status: "idle" as const, // "idle" literal type으로 유지
}));
```
또는 전체 객체를 고정할 수도 있습니다:
```ts
const useStore = create(() => ({
  status: "idle",
  count: 0,
} as const));
```
그러면 `status: "idle"` 및 `count: 0`으로 추론되며, 정확한 리터럴 타입이 유지됩니다.
> ⚠️단점: `as const`를 쓰면 모든 속성이 `readonly`가 되므로 상태 변경이 필요한 부분에서는 주의해야 합니다.

## Redux action Type에서 widening 제어
Redux 액션에서 type 필드를 리터럴 타입으로 정확히 지정하는 것이 매우 중요합니다. 그렇지 않으면 액션 디스패치 시에 타입 좁히기가 제대로 안 됩니다.

### ❌ widening된 액션 타입 (문제 발생)
```ts
const incrementAction = {
  type: "INCREMENT"
};
// 추론: { type: string }
```
- `"INCREMENT"`가 string으로 widen됨.
- 타입 가드에서 `"INCREMENT"`를 검사해도 타입이 좁혀지지 않음.

### `as const`로 방지
```ts
const incrementAction = {
  type: "INCREMENT"
} as const;
// 추론: { readonly type: "INCREMENT" }
// discriminated union
type Action =
  | { type: "INCREMENT" }
  | { type: "DECREMENT" };

function reducer(state: number, action: Action) {
  switch (action.type) {
    case "INCREMENT":
      return state + 1; // OK
    case "DECREMENT":
      return state - 1; // OK
  }
}
```

## 🔧 대안: 명시적 타입 지정
`as const`를 쓰지 않더라도 타입을 명시해도 widening을 방지할 수 있습니다:
```ts
const incrementAction: { type: "INCREMENT" } = {
  type: "INCREMENT"
};
```
- 하지만 이건 타입 중복이 생기므로 `as const`를 선호하는 경우가 많습니다.

# 타입 좁히기 Narrowing (타입 축소)
타입 시스템의 핵심 기능 중 하나이며, 복잡한 조건문에서도 정확한 타입 추론을 가능하게 해줍니다.
## 💡 Narrowing이란?
```ts
function printLength(val: string | number) {
  if (typeof val === "string") {
    // val: string
    console.log(val.length);
  } else {
    // val: number
    console.log(val.toFixed(2));
  }
}
```
**Narrowing**은 변수의 **유니언 타입**(e.g. string | number)에서 **조건문 등을 통해 더 좁은 타입으로 추론하는 과정**입니다.
### Control Flow Analysis (CFA)
- TypeScript는 코드의 제어 흐름을 따라가며 변수의 타입을 분석함.
- 이를 통해 블록 내에서 가장 구체적인 타입으로 자동 추론합니다.

## 🔎 주요 타입 가드 메커니즘
타입 가드는 타입을 좁혀주는 조건식입니다. 대표적인 가드 메커니즘:
| 종류                | 예시                         | 설명                                                                          |
| ----------------- | -------------------------- | --------------------------------------------------------------------------- |
| `typeof`          | `typeof x === "string"`    | 기본 타입 (`string`, `number`, `boolean`, `object`, `function`, `undefined`) 구분 |
| `instanceof`      | `x instanceof Date`        | 클래스 인스턴스 여부 판별                                                              |
| `'prop' in obj`   | `'name' in person`         | 특정 속성 존재 여부로 타입 분기                                                          |
| `Array.isArray()` | `Array.isArray(x)`         | 배열 여부 확인                                                                    |
| 사용자 정의 타입 가드      | `isFish(pet): pet is Fish` | 아래 6번에서 설명                                                                  |

## ⚖️ Truthy / Falsy 기반 Narrowing
조건문에서의 truthy/falsy 체크로도 narrowing이 가능합니다.
```ts
function greet(name?: string) {
  if (name) {
    // name: string
    console.log("Hello, " + name.toUpperCase());
  } else {
    // name: undefined
    console.log("Hello, guest");
  }
}
```
| 값                                              | Falsy | Truthy |
| ---------------------------------------------- | ----- | ------ |
| `false`, `0`, `''`, `null`, `undefined`, `NaN` | ✔     | ✘      |

## 🧩 Discriminated Union을 활용한 Narrowing
유니언 타입의 공통 literal 속성을 활용해 타입을 좁히는 방식입니다. 가장 직관적이며 추천되는 형태입니다.
```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      // shape: { kind: "circle", radius: number }
      return Math.PI * shape.radius ** 2;
    case "square":
      // shape: { kind: "square", side: number }
      return shape.side ** 2;
  }
}
```
- kind 같은 공통 literal 속성을 **Discriminant (판별자)**라고 합니다.
- 가장 효과적이고 안정적인 narrowing 방식입니다.

## 🚨 Exhaustiveness Check와 never의 역할
**Discriminated Union을 switch 문으로 처리할 때**, 모든 타입을 빠짐없이 다뤘는지 확인하려면 `never`와 default case를 이용합니다.
```ts
function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      throw new Error(`Unhandled shape: ${shape}`);
  }
}
```
- 만약 새로운 타입을 추가했지만 `switch`에 안 넣으면 **컴파일 에러 발생 → 안전하게 리팩토링 가능**
- `never`은 절대 도달할 수 없는 타입을 의미하며, 이를 통해 **누락된 케이스를 컴파일 타임에 검출**합니다

## 🛠 사용자 정의 타입 가드 실습
```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim(); // pet: Fish
  } else {
    pet.fly(); // pet: Bird
  }
}
```
### 사용자 정의 타입 가드의 장점
- 코드 재사용성 향상
- 조건문 내에서 정확한 타입 추론
- 복잡한 구조에서 특히 유용


### 🔹 Narrowing (타입 축소)
조건문 등의 문맥을 통해 더 구체적인 타입으로 좁히는 과정
```ts
function print(value: string | number) {
  if (typeof value === 'string') {
    // value: string
    console.log(value.toUpperCase());
  } else {
    // value: number
    console.log(value.toFixed(2));
  }
}
```

# 타입스크립트의 복잡한 케이스와 컴파일러 동작
## 조건부 타입과의 연관
조건부 타입은 제네릭 기반의 타입 논리를 분기할 수 있게 해줍니다:
```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false
```
### 분배 조건부 타입 (Distributive Conditional Types)
```ts
type ToArray<T> = T extends any ? T[] : never;
type A = ToArray<string | number>; // string[] | number[]
```
- `T extends any ? ...` 은 유니언 타입에 대해 **분배(distribution)**됨
- 즉, `ToArray<string | number>`는 `ToArray<string> | ToArray<number>`로 동작함
### 좁히기와의 관계
조건부 타입 내에서 T가 좁혀지지 않는 경우가 있습니다:
```ts
function test<T>(val: T) {
  type R = T extends string ? "string" : "other";
  // 여기서 T가 실제 string인지 실행 시점엔 모름 → 타입 좁히기 불가
}
```
- 조건부 타입은 선언 시점의 타입(T)에 대해만 작동하며, 런타임 조건에는 반응하지 않음

## 🧬 제네릭 사용 시 widening/narrowing 컨트롤
제네릭을 사용할 때 타입이 widening될 수도 있고, 좁히기 실패가 발생할 수 있습니다.
### 문제 예시
```ts
function wrap<T>(x: T) {
  return { value: x };
}

const wrapped = wrap("hello"); 
// wrapped.value: string ("hello" → widening)
```
`as const`로 리터럴 유지:
```ts
const wrapped = wrap("hello" as const);
// wrapped.value: "hello"
```
또는 제네릭 파라미터에 `T extends string`을 제한할 수도 있습니다.
```ts
function process<T extends "start" | "stop">(state: T) {
  if (state === "start") {
    // state: "start"로 좁혀짐 (narrowing OK)
  }
}
```
> 💡 문제는 T가 string 전체로 추론될 때 narrowing이 실패한다는 것 → 제네릭 타입의 범위를 제한하거나, 리터럴 타입으로 고정하는 방식으로 제어해야 함.

## 🧭 keyof, Mapped Types에서의 타입 흐름 추적
```ts
type User = { name: string; age: number };
type Keys = keyof User; // "name" | "age"

type StringProps<T> = {
  [K in keyof T]: T[K] extends string ? K : never
}[keyof T];
// "name"
```
### 타입 흐름 설명:
1. Mapped Type: `[K in keyof T]`로 모든 키 순회.
1. 각 키에 대해 조건부 타입 적용: `T[K] extends string ? K : never`
1. 최종적으로 `[keyof T]`를 통해 유니언 타입으로 병합됨.
📌 주의할 점: 위 과정에서 리터럴 타입이 widening되면, 조건부 분기가 제대로 작동하지 않을 수 있습니다. 예: string vs "literal".

## 🔄 ReturnType<T> 등 유틸 타입에서 widening이 개입되는 시점
```ts
function makeUser() {
  return {
    name: "Alice",
    age: 30,
  };
}

type User = ReturnType<typeof makeUser>;
// name: string, age: number
```
- `"Alice"` → `"string"`으로, `30` → `number`로 widening됨.
- 즉, `ReturnType`은 함수 반환 타입을 그대로 추론하지만, 리터럴 값들은 기본적으로 widened 상태로 포함됨.
```ts
function makeUser() {
  return {
    name: "Alice",
    age: 30,
  } as const;
}

type User = ReturnType<typeof makeUser>;
// name: "Alice", age: 30
```
## 🚫 타입 좁히기 실패 사례와 해결 방법
실패 예시 1: `T`가 유니언일 때 narrowing 안 되는 경우
```ts
function handle<T extends "a" | "b">(value: T) {
  if (value === "a") {
    // TypeScript는 여기서 T = "a"로 좁히지 않음 (일부 경우만 작동)
  }
}
```
- ✅ 해결: 리터럴 값 전달 시, 구체 타입으로 고정하거나 오버로드 사용

실패 예시 2: 객체 프로퍼티 기반 narrowing 실패
```ts
type Data =
  | { type: "text"; value: string }
  | { type: "number"; value: number };

function process(data: Data) {
  if (data.type === "text") {
    // OK: data.value → string
  } else if (data.type === "number") {
    // OK: data.value → number
  } else {
    // ❌ 타입 오류 안 남 → exhaustiveness check 실패
  }
}

// 해결안
function process(data: Data) {
  switch (data.type) {
    case "text":
      break;
    case "number":
      break;
    default:
      const _exhaustive: never = data; // 컴파일 에러 발생 시 누락 알림
  }
}
```

## 요약 정리
| 항목           | 설명                          | 위험/주의                            |
| ------------ | --------------------------- | -------------------------------- |
| 조건부 타입       | `T extends string ? A : B`  | 런타임 narrowing 불가                 |
| 제네릭          | 리터럴이 일반 타입으로 widening될 수 있음 | `as const`, 범위 제한                |
| mapped types | `keyof`, 조건부 타입 결합으로 흐름 분석  | 중간 widening 주의                   |
| ReturnType   | 함수 반환 타입 추론 시 widening 발생   | `as const`로 값 고정                 |
| narrowing 실패 | 제네릭 또는 분기 누락                | `never` 활용한 exhaustiveness check |

# 실무 예제
## useState 초기화에서 widening 오류
```ts
const [status, setStatus] = useState("idle"); 
// status: string (❌ widened from "idle")

if (status === "loading") {
  // ❌ TS 오류: status는 string이고, 그 중 loading인지 알 수 없음
}

//해결책
const [status, setStatus] = useState<"idle" | "loading" | "success">("idle");
// or
const [status, setStatus] = useState(() => "idle" as const);
```
- `as const`는 `status` 초기값을 `"idle"` 리터럴로 유지시켜 widening 방지
- `useState<리터럴 유니언>`을 직접 명시하는 것도 안전한 방법

## API 응답 타입 좁히기 실패
```ts
type ApiResponse =
  | { success: true; data: { userId: string } }
  | { success: false; error: string };

function handleResponse(res: ApiResponse) {
  if (res.success) {
    // res: still ApiResponse (❌ narrowing 실패)
    console.log(res.data.userId); // 오류
  }
}
```
**해결방법**
TypeScript는 `res.success`가 `true`라는 조건을 **명확히 narrow**하지 못함. 이럴 때는 **Discriminated Union + Exhaustiveness Check** 사용:
```ts
function handleResponse(res: ApiResponse) {
  if (res.success === true) {
    // ✅ OK: res는 { success: true; data: ... }
    console.log(res.data.userId);
  } else {
    console.error(res.error);
  }
}
```
또는 `res.success`를 `boolean`으로 추론했다면 타입 강제 필요.
## 리팩토링: 타입 보존을 위한 리터럴 고정 전략
### 핵심 전략
| 상황           | 전략                                             |
| ------------ | ---------------------------------------------- |
| 함수 반환 객체     | `as const`로 리터럴 유지                             |
| useState     | 제네릭으로 리터럴 유니언 타입 지정                            |
| Redux action | `payload: T` 형태로 제네릭 분리                        |
| 객체 초기화       | `as const` 사용                                  |
| 함수 인자 리터럴    | `"value" as const` 또는 제네릭 제한 (`T extends ...`) |
### 예시: 버튼 상태를 관리하는 Hook
```ts
const [mode, setMode] = useState<"read" | "write">("read");
// 또는
const [mode, setMode] = useState(() => "read" as const);
```

## `satisfies` 연산자 활용 (TS 4.9+)
```ts
const config = {
  mode: "light",
  version: "1.0",
} satisfies {
  mode: "light" | "dark";
  version: string;
};

// config.mode: "light" (✔ 리터럴 유지됨)
```
> `satisfies`는 타입 검증은 받되, 값은 그대로 보존 → widening 방지 + 추론 정확도 증가

## 요약
| 주제                      | 요점                        | 해결 방법                          |
| ----------------------- | ------------------------- | ------------------------------ |
| useState widening 문제    | "idle"이 string으로 추론됨      | 제네릭 명시 or `as const`           |
| API 타입 좁히기 오류           | 조건식이 모호하거나 넓음             | discriminated union + 리터럴 체크   |
| 리터럴 보존 전략               | 객체, 인자, 리턴값 등에서 리터럴 유지    | `as const`, `T extends`, 타입 명시 |
| satisfies (TS 4.9+)     | 구조적 타입 검사는 하되 widening 방지 | `satisfies` 연산자 사용             |

