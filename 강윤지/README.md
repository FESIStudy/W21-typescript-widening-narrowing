## 1. as const와 리터럴 타입 고정

- 타입스크립트에서 **값을 "리터럴 타입"으로 고정(freeze)** 시켜주는 **단언(assertion)** 문법.
- 예시
    
    ```tsx
    const greeting = "hello";
    // greeting: string
    ```
    
    ```tsx
    const greeting = "hello" as const;
    // greeting: "hello" (리터럴 타입)
    ```
    
    → `as const`가 없으면 `"hello"`라는 값을 `string`으로 넓혀서 추론하지만, `as const`를 사용하면 **정확히 `"hello"`라는 값**으로 타입이 고정된다.
    
- 목적: 리터럴 타입으로 고정해야 **엄격한 타입 검증**이 가능하다.
    
    ```tsx
    const INCREMENT = "INCREMENT";
    const action = { type: INCREMENT };
    
    // 타입스크립트는 action.type을 string으로 봄 (너무 넓음)
    ```
    
    ```tsx
    const INCREMENT = "INCREMENT" as const;
    const action = { type: INCREMENT };
    // action.type: "INCREMENT" (정확한 리터럴 타입)
    ```
    
    - 객체와 배열에서도 사용 가능
        
        ```tsx
        const status = {
          loading: true,
          message: "Fetching data",
        } as const;
        ```
        
        - 이 객체는 다음과 같이 타입 고정됨:
        
        ```tsx
        {
          readonly loading: true;
          readonly message: "Fetching data";
        }
        ```
        
        즉, **값이 고정**되고 **모든 프로퍼티는 읽기 전용(readonly)**이 된다.
        

## 2. enum vs as const 차이점

| 구분 | `enum` | `as const` |
| --- | --- | --- |
| **런타임 코드** | JS로 컴파일됨 (코드 증가) | 값 자체로만 존재 (추가 코드 없음) |
| **타입 정의 방식** | 타입과 값 동시에 생성됨 | 값(`object`)에 `typeof`로 타입 추출 |
| **확장성** | 명시적으로 선언된 값만 가능 | 객체 확장/조합 가능 |
| **문자열 매핑** | 기본적으로 `number` 기반 (string 지정 가능) | key-value로 직접 명시 |
| **추천 시점** | 복잡한 enum 로직, 상수 집합이 필요할 때 | 간단한 리터럴 집합, 타입 기반 추론에 유리 |
| **Tree-shaking** | 잘 안 됨 (런타임 코드 존재) | 매우 잘 됨 (값만 남음) |
| **React 등 실무 사용** | 요즘 잘 안 씀 (JS 코드 증가) | 매우 자주 씀 (`as const` + `typeof` 조합) |

### 2-1. enum 방식

```tsx
enum Direction {
  Up = "UP",
  Down = "DOWN",
}

function move(dir: Direction) {
  if (dir === Direction.Up) {
    console.log("Going up");
  }
}
```

> 컴파일 결과:
> 

```tsx
"use strict";
var Direction;
(function (Direction) {
  Direction["Up"] = "UP";
  Direction["Down"] = "DOWN";
})(Direction || (Direction = {}));
```

> 런타임 코드가 생김 → 번들 크기에 영향 있음
> 

### 2. `as const` 방식

```tsx
const Direction = {
  Up: "UP",
  Down: "DOWN",
} as const;

type Direction = typeof Direction[keyof typeof Direction]; // "UP" | "DOWN"

function move(dir: Direction) {
  if (dir === Direction.Up) {
    console.log("Going up");
  }
}
```

> 컴파일 결과:
> 

```tsx
const Direction = {
  Up: "UP",
  Down: "DOWN"
};
```

> ✅ 런타임 코드 거의 없음 → 타입 시스템에서만 사용
> 

---

## 3. 타입 추론 vs 타입 단언

### 3-1. 타입 추론 (Type Inference)

- 개념: TypeScript가 변수나 함수 등의 타입을 **자동으로 유추**하려는 것을 말한다.

```tsx
let message = "hello";
// 추론된 타입: string

const nums = [1, 2, 3];
// 추론된 타입: number[]
```

- 사용: 변수 선언 시, 함수 반환 타입, 매개변수 기본값, 구조 분해 할당

```tsx
function add(x = 1, y = 2) {
  return x + y;
}
// 반환 타입: number (자동 추론됨)
```

- 장점: 코드가 간결하고 읽기 쉬움
- 단점: 복잡한 상황에서는 추론이 잘못되거나 너무 넓어질 수 있음
- 추론보다 명시가 나은 경우

```tsx
function fetchData(): Promise<User[]> {
  // API 호출...
}
```

- **함수/모듈의 외부 인터페이스**에서는 타입을 명시하는 게 좋다.
- **자동 추론에 의존하면 의도가 불명확**할 수 있음.

### 3-2. 타입 단언 (Type Assertion)

- 개념: 개발자가 “**내가 타입을 확실히 알고 있으니, TypeScript야, 그냥 믿어**”라고 선언하는 것. 즉, **컴파일러에게 타입을 단언하는 행위**

```tsx
const el = document.querySelector("#app") as HTMLDivElement;
el.innerText = "hello";
```

- 사용 방식 - 2가지

```tsx
const el = value as HTMLDivElement; // JSX와의 충돌 방지를 위해 실무에서는 as 문법 권장
const el = <HTMLDivElement>value; // JSX 환경에서는 사용 지양
```

- 주의점
    - 단언은 타입을 강제로 덮어쓰는 것이기에, 런타임 오류로 이어질 수 있음
    
    ```tsx
    const el = document.querySelector("#app") as HTMLInputElement;
    el.value = "hi"; // ❌ 실제로는 div일 수도 있음 → 런타임 에러!
    ```
    
    → 단언은 컴파일 시 오류를 막지만, 실제 안전을 보장하지 않음!
    
- 단언이 필요한 상황들
    
    1. `querySelector` 또는 DOM 조작
    
    ```tsx
    const input = document.querySelector("input") as HTMLInputElement;
    input.value = "hello";
    ```
    
    2. `JSON.parse()` 등 반환 타입이 `any`인 경우
    
    ```tsx
    const user = JSON.parse(data) as { name: string; age: number };
    ```
    
    3. `ref`, `forwardRef`와 같이 타입 추론이 어렵거나 깨지는 경우
    
    ```tsx
    const myRef = useRef<HTMLDivElement>(null);
    myRef.current?.focus();
    ```
    
- 단언 vs 추론 비교

| 상황 | 코드 | 설명 |
| --- | --- | --- |
| 자동 추론 | `let x = 1;` | `x: number`로 추론됨 |
| 명시적 타입 | `let x: number = 1;` | 타입을 강제 지정 |
| 단언 | `let x = "hi" as number;` | 타입스크립트는 오류 없음, 런타임에서는 문제 발생 가능 |
- 단언 대신 사용할 수 있는 안전한 방법
    - 사용자 정의 타입 가드
    
    ```tsx
    function isInput(el: Element): el is HTMLInputElement {
      return el.tagName === "INPUT";
    }
    
    const el = document.querySelector("#myInput");
    if (el && isInput(el)) {
      el.value = "hello"; // 안전하게 접근
    }
    ```
    

## 4. 타입 넓히기(Widening)와 좁히기(Narrowing)

**넓히기**는 타입이 "일반화"되는 현상, **좁히기**는 타입이 "구체화"되는 과정

### 4-1. 타입 넓히기 (Widening)

- 개념: **초기값으로 지정된 리터럴 값**이 더 **일반적인 타입으로 추론되는 현상.** TypeScript가 안전하고 유연한 코드를 위해 자동으로 타입을 일반화함

```tsx
let greeting = "hello"; // 타입: string (넓혀짐)
const greeting2 = "hello"; // 타입: "hello" (리터럴 타입)
```

| 선언 방식 | 타입 |
| --- | --- |
| `let greeting = "hi"` | `string` (넓힘) |
| `const greeting = "hi"` | `"hi"` (리터럴 고정) |
| `let greeting: "hi" = "hi"` | `"hi"` (명시적으로 고정) |
- Widening Literal Types
    
    ```tsx
    let count = 1;      // count: number
    const count2 = 1;   // count2: 1
    ```
    
    - `let`은 리터럴 값을 일반 타입으로 넓힘
    - `const`는 **리터럴 타입으로 고정됨**
    
    ```tsx
    let theme = "light"; // string
    const mode = "light"; // "light"
    ```
    
    → 실무 팁: 리터럴 타입을 유지하고 싶다면 as const 또는 const를 적극 활용하자
    

### 4-2. 타입 좁히기 (Narrowing)

- 개념: **유니언 타입(union type)**을 다룰 때, **조건문 등을 이용해 구체적인 타입으로 줄여나가는 과정.** 즉, 실행 흐름에 따라 타입을 판별하고 줄이는 것

```tsx
function print(value: string | number) {
  if (typeof value === "string") {
    // value: string
    console.log(value.toUpperCase());
  } else {
    // value: number
    console.log(value.toFixed(2));
  }
}
```

- 자주 사용하는 Narrowing 기법

| 방법 | 예시 | 설명 |
| --- | --- | --- |
| `typeof` 검사 | `typeof value === "string"` | 원시 타입 구분 |
| `instanceof` 검사 | `value instanceof Date` | 클래스 기반 타입 구분 |
| `in` 연산자 | `"bark" in animal` | 특정 프로퍼티 존재 여부로 구분 |
| 태그 유니언 (`discriminated union`) | `if (shape.type === "circle")` | 공통 프로퍼티로 타입 구분 |
| 사용자 정의 타입 가드 | `isString(val): val is string` | 커스텀 타입 구분 함수 |
- 예시: `in` 연산자

```tsx
type Dog = { bark: () => void };
type Cat = { meow: () => void };

function speak(animal: Dog | Cat) {
  if ("bark" in animal) {
    animal.bark(); // Dog
  } else {
    animal.meow(); // Cat
  }
}
```

- 예시: Discriminated Union

```tsx
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; size: number };

function area(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  }
  return shape.size ** 2;
}
```

- 사용자 정의 타입 가드

```tsx
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function printLength(value: unknown) {
  if (isString(value)) {
    console.log(value.length); // 안전!
  }
}
```

- 주의할 점
    - **넓혀진 타입은 리터럴 비교나 조건 분기에서 정보가 손실될 수 있음**
        
        ```tsx
        let button = "primary";
        type Variant = "primary" | "secondary";
        const variant: Variant = button; // ❌ string 타입이므로 에러
        ```
        
        해결법:
        
        ```tsx
        const button = "primary" as const;
        type Variant = "primary" | "secondary";
        const variant: Variant = button; // ✅ OK
        ```
        

## 5. 유틸리티 타입(Utility Types)

이미 존재하는 타입을 바탕으로 **새로운 타입을 쉽게 변형**할 수 있게 도와주는 타입 도우미들.

 TypeScript의 표준 라이브러리에 기본 내장된 제네릭 타입이며, `type`이나 `interface`를 더 유연하고 효율적으로 다룰 수 있게 해준다.

### 5-1. Partial<T>

> 모든 프로퍼티를 **선택적(optional)**으로 바꾼다.
> 

```tsx
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// 결과:
// {
//   id?: number;
//   name?: string;
//   email?: string;
// }
```

→ API 응답값에서 활용 가능.

### 5-2. Required<T>

> 모든 프로퍼티를 **필수(required)**로 바꾼다.
> 

```tsx
interface Props {
  title?: string;
  description?: string;
}

type FullProps = Required<Props>;
// {
//   title: string;
//   description: string;
// }
```

### 5-3. Readonly<T>

> 모든 프로퍼티를 **읽기 전용(readonly)**으로 바꾼다.
> 

```tsx
interface Todo {
  id: number;
  text: string;
}

type ReadonlyTodo = Readonly<Todo>;
// {
//   readonly id: number;
//   readonly text: string;
// }

const todo: ReadonlyTodo = { id: 1, text: "study" };
todo.id = 2; // ❌ 오류: 읽기 전용 속성
```

### 5-4. Pick<T, K>

> T 타입에서 일부 키 K만 골라서 새로운 타입을 만든다.
> 

```tsx
interface Post {
  id: number;
  title: string;
  content: string;
}

type PostPreview = Pick<Post, "id" | "title">;
// {
//   id: number;
//   title: string;
// }
```

### 5-5. Omit<T, K>

> T 타입에서 특정 키 K를 제외하고 타입을 만든다.
> 

```tsx
type PostBody = Omit<Post, "id">;
// {
//   title: string;
//   content: string;
// }
```

### 5-6. Record<K, T>

> K 타입의 키들을 가지며, 각 키의 값은 T 타입인 객체를 생성한다.
> 

```tsx
type Role = "admin" | "user" | "guest";

const permissions: Record<Role, boolean> = {
  admin: true,
  user: true,
  guest: false,
};
```

### 5-7. Exclude<T, U>

> T에서 U에 해당하는 타입을 제거한다.
> 

```tsx
type Status = "loading" | "success" | "error";
type NonErrorStatus = Exclude<Status, "error">;
// "loading" | "success"
```

### 5-8. Extract<T, U>

> T에서 U에 해당하는 타입만 남긴다.
> 

```tsx
type OnlyError = Extract<Status, "error">;
// "error"
```

### 5-9. NonNullable<T>

> null과 undefined를 제외한 타입을 만든다.
> 

```tsx
type MaybeUser = string | null | undefined;
type SafeUser = NonNullable<MaybeUser>; // string
```

### 5-10. ReturnType<T>

> 함수 타입 T의 반환값 타입을 추출한다.
> 

```tsx
function getUser() {
  return { id: 1, name: "Alice" };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string }
```

---

### 5-11. Parameters<T>

> 함수 타입 T의 인자 타입을 튜플로 추출한다.
> 

```tsx
function greet(name: string, age: number): void {}

type GreetParams = Parameters<typeof greet>;
// [name: string, age: number]
```

### 5-12. ConstructorParameters<T>

> 생성자(constructor) 함수의 인자 타입을 추출한다.
> 

```tsx
class Person {
  constructor(public name: string, public age: number) {}
}

type PersonArgs = ConstructorParameters<typeof Person>;
// [name: string, age: number]
```

---

### 5-13. InstanceType<T>

> 생성자 함수 타입 T로부터 생성된 인스턴스의 타입을 얻는다.
> 

```tsx
type PersonInstance = InstanceType<typeof Person>;
// Person
```

---

## ✅ 유틸리티 타입 요약표

| 유틸리티 | 설명 |
| --- | --- |
| `Partial<T>` | 모든 속성을 optional로 만듦 |
| `Required<T>` | 모든 속성을 필수로 만듦 |
| `Readonly<T>` | 모든 속성을 readonly로 만듦 |
| `Pick<T, K>` | T에서 K 속성만 선택 |
| `Omit<T, K>` | T에서 K 속성 제외 |
| `Record<K, T>` | K를 key로, T를 value로 하는 객체 생성 |
| `Exclude<T, U>` | T에서 U를 제거 |
| `Extract<T, U>` | T에서 U만 추출 |
| `NonNullable<T>` | null과 undefined 제거 |
| `ReturnType<T>` | 함수의 반환 타입 추출 |
| `Parameters<T>` | 함수 인자 타입 튜플 추출 |
| `ConstructorParameters<T>` | 생성자 인자 타입 추출 |
| `InstanceType<T>` | 생성자 인스턴스 타입 추출 |
