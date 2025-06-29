## 평소 궁금했던 타입스크립트 문법들 살펴보기

### let func: <T, U>(a: T, d: U) => U = (a, d) => d;

간단하다 위 코드를 `let func: <T, U>(a: T, d: U) => U` // `= (a, d) => d;` 로 쪼개서 보면 되는데
앞에 부분은 함수의 타입선언부분으로 T,U를 인자로 받아 U를 리턴하는 함수라는 의미이고
뒷 부분은 함수의 할당부분으로 실제 구현부를 func에 할당한것.

그럼 질문

```ts
function snd(a: T, d: U): U {
  return d;
}
```

이거랑은 뭐가 다른가.
완전히 다르다.

후자는 외부에서 T,U가 타입선언이 되어있지 않으면 에러가 발생하고 전자는 발생하지않는다.
함수 내부에서 T,U를 어댑티브하게 타입을 선언했다고 생각하면 이해가 쉽다.

### 유틸리티 타입

이미 선언된 타입을 필요에 따라 매번 새로 선언할 수 없으니 그떄그때 가변적으로 타입을 적용시키기 위해 사용되는 타입변형

알겠습니다!
대상이 **3년 차 웹개발자**라면 이미 자바스크립트에 익숙하고 타입스크립트도 기본적으로 활용 중일 가능성이 높기 때문에, 이 글에서는 **"실무에서 자주 마주치는 상황에 유용한 유틸리티 타입들"을 중심으로** 소개하겠습니다.

---

# 🛠 TypeScript 유틸리티 타입 제대로 알기

> 실무에 당장 써먹을 수 있는 타입 레벨 도구 모음

---

## 🧩 유틸리티 타입이란?

타입스크립트에서는 반복적으로 작성하게 되는 타입 변형을 \*\*함수처럼 처리할 수 있는 "타입 도구"\*\*를 제공합니다.
이걸 \*\*유틸리티 타입(Utility Types)\*\*이라고 부르며, 대부분 TypeScript에 내장되어 있습니다.

예:

### 2. `Required<T>`

**→ 모든 프로퍼티를 필수로 만듭니다**

```ts
type Draft = { id?: number; name?: string };
type User = Required<Draft>;
// { id: number; name: string }
```

---

### 3. `Readonly<T>`

**→ 모든 필드를 읽기 전용으로 만듭니다**

```ts
type Config = Readonly<{ port: number }>;
// 수정 시 에러: config.port = 3000;
```

---

### 4. `Pick<T, K>`

**→ 객체에서 일부 프로퍼티만 뽑아냅니다**

```ts
type User = { id: number; name: string; email: string };
type Preview = Pick<User, 'id' | 'name'>;
// { id: number; name: string }
```

> 💡 대형 객체에서 UI 컴포넌트에 필요한 필드만 골라낼 때 유용

---

### 5. `Omit<T, K>`

**→ 특정 프로퍼티를 제외합니다**

```ts
type WithoutEmail = Omit<User, 'email'>;
// { id: number; name: string }
```

> 💡 서버 응답에서 민감 정보 제거할 때 자주 사용

---

### 6. `Record<K, T>`

**→ 특정 key 목록을 기반으로 매핑된 객체 타입을 생성**

```ts
type Roles = 'admin' | 'guest' | 'editor';
type RoleFlags = Record<Roles, boolean>;
// { admin: boolean; guest: boolean; editor: boolean }
```

> 💡 enum-like 구조를 타입 레벨에서 다룰 때 아주 유용

---

### 7. `Exclude<T, U>`

**→ 유니언 타입에서 U를 제외합니다**

```ts
type Roles = 'admin' | 'guest' | 'editor';
type NonGuests = Exclude<Roles, 'guest'>;
// 'admin' | 'editor'
```

---

### 8. `Extract<T, U>`

**→ 유니언 타입에서 U만 남깁니다**

```ts
type Event = 'click' | 'hover' | 'scroll';
type UIEvent = Extract<Event, 'click' | 'hover'>;
// 'click' | 'hover'
```

---

### 9. `NonNullable<T>`

**→ null과 undefined를 제거**

```ts
type MaybeString = string | null | undefined;
type SafeString = NonNullable<MaybeString>; // string
```

---

### 10. `ReturnType<T>`

**→ 함수 타입에서 반환값의 타입만 추출**

```ts
function getUser() {
  return { id: 1, name: 'Alice' };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string }
```

## 타입가드란?

타입스크립트를 사용해 개발하다보면 하나의 값이 두개의 타입이 유니온으로 묶여있을때가 있는데
이때 서로의 타입의 키 값이 서로 다르다고 할때 어떤 한 타입의 한 키값을 참조해야하는 상황이라면
바로 타입가드가 필요하다.

```tsx
import { createFileRoute } from '@tanstack/react-router';

import { useCurrentRequestQueue } from '@/hooks/current/useCurrentRequestQueue';
import type { RequestQueue } from '@/types/extension';
import type { BitcSwitchNetwork } from '@/types/message/inject/bitcoin';

import Entry from './-entry';
import Layout from './-layout';
import AccessRequest from '../../-components/requests/AccessRequest';

export const Route = createFileRoute('/popup/bitcoin/switch-network/')({
  component: BitcoinSwitchChain,
});

function BitcoinSwitchChain() {
  const { currentRequestQueue } = useCurrentRequestQueue();

  if (currentRequestQueue && isBitcoinSwitchNetwork(currentRequestQueue)) {
    return (
      <AccessRequest>
        <Layout>
          <Entry request={currentRequestQueue} />
        </Layout>
      </AccessRequest>
    );
  }
  return null;
}

function isBitcoinSwitchNetwork(queue: RequestQueue): queue is BitcSwitchNetwork {
  return queue.method === 'bitc_switchNetwork';
}
```

위 코드에서 Entry컴포넌트에 currentRequestQueue가 BitcSwitchNetwork임을 확신하게 하도록 isBitcoinSwitchNetwork 를 사용하여 타입가드를 사용한 모습이다.

## 제네릭 타입

좋습니다. 이번엔 **제네릭(Generic) 타입**에 대해 설명드릴게요.
대상은 **3년 차 웹개발자**이므로 이미 `any`나 `unknown` 같은 타입으로 유연하게 처리해본 경험이 있다고 보고,
\*\*“왜 any보다 제네릭이 더 나은가?”, “실무에서 어떻게 쓰이는가?”\*\*를 중심으로 정리하겠습니다.

---

# 📦 타입스크립트의 제네릭(Generic) 타입이란?

> **제네릭은 타입에 이름을 붙여서, 나중에 사용할 수 있게 만든 변수 같은 것입니다.**

즉, 함수나 클래스, 타입에서 **타입을 나중에 정하자**는 거예요.

```ts
function identity<T>(value: T): T {
  return value;
}
```

이 함수는 `T`가 어떤 타입인지 미리 정하지 않아요.
함수를 호출할 때 전달된 타입을 **자동으로 추론하거나 명시적으로 지정**합니다.

```ts
identity<number>(123); // T는 number
identity('hello'); // T는 string
```

---

## ❗왜 제네릭을 쓰는가?

### `any`와 비교해볼게요:

```ts
function wrapAny(value: any) {
  return value;
}

function wrapGeneric<T>(value: T): T {
  return value;
}
```

`wrapAny`는 아무 타입을 받아도 되지만, 반환값도 **타입 안전성을 잃어버립니다**:
타입 안정성을 잃어버린다가 포인트라고 생각함.

```ts
const result = wrapAny(123); // any → result.toFixed() 가능? 오류 안 나도 위험
```

반면 `wrapGeneric`은 전달된 타입을 기억하기 때문에:

```ts
const result = wrapGeneric(123); // number → result.toFixed() 안전하게 인식됨
```

> ✅ 즉, **유연함을 유지하면서 타입 안정성을 가져가는 방법**이 바로 제네릭입니다.

---

## 🔧 실무에서 자주 쓰이는 제네릭 패턴들

---

### 1. **배열 요소 타입 유지**

```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

const x = first(['a', 'b', 'c']); // string
const y = first([1, 2, 3]); // number
```

→ 배열에서 꺼낸 값을 정확하게 추론할 수 있음

---

### 2. **컴포넌트 Props에 제네릭 사용 (React)**

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
};

function List<T>({ items, renderItem }: ListProps<T>) {
  return <>{items.map(renderItem)}</>;
}

// 사용 예
<List items={[1, 2, 3]} renderItem={(i) => <div>{i}</div>} />;
```

→ 다양한 타입의 리스트를 하나의 컴포넌트로 처리 가능

---

### 3. **API 응답 타입 통일**

```ts
type ApiResponse<T> = {
  status: number;
  data: T;
};

const res: ApiResponse<{ name: string }> = {
  status: 200,
  data: { name: 'Alice' },
};
```

→ 여러 API의 결과를 **하나의 제네릭 타입으로 통일**할 수 있음

---

### 4. **함수에서 조건 기반으로 타입 유지**

```ts
function pickKey<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: 'Shawn' };
const name = pickKey(user, 'name'); // string으로 추론됨
```

## 📌 제네릭 제약조건 (`extends`)

제네릭을 완전히 열어두면 너무 위험할 수도 있어서 **제약을 둘 수 있어요**:

```ts
function getLength<T extends { length: number }>(val: T): number {
  return val.length;
}

getLength([1, 2, 3]); // OK
getLength('hello'); // OK
getLength(123); // ❌ number에는 length 없음
```

## never타입이란?

> `never`는 **절대 발생하지 않는 값**을 나타내는 타입입니다.

즉, **"값이 존재하지 않는다"는 걸 타입 레벨에서 표현하는 타입**이에요.

---

## 🧠 비슷한 개념들과 비교

| 타입        | 의미                                                            |
| ----------- | --------------------------------------------------------------- |
| `undefined` | 값이 정의되지 않음 (값은 있음)                                  |
| `null`      | 명시적으로 비어 있음 (값은 있음)                                |
| `void`      | 함수가 아무것도 반환하지 않음                                   |
| `never`     | **아예 어떤 값도 존재할 수 없음** ← 여기서만 에러, 종료 등 발생 |

---

## 🔥 언제 `never`가 등장할까?

### 1. **항상 예외를 던지는 함수**

```ts
function throwError(): never {
  throw new Error('Something went wrong');
}
```

→ 이 함수는 절대 정상적으로 반환되지 않기 때문에 \*\*반환 타입이 `never`\*\*입니다.

---

### 2. **무한 루프**

```ts
function loopForever(): never {
  while (true) {}
}
```

→ 이 함수는 끝날 수 없으므로 `never`로 추론됩니다.

---

### 3. **타입 좁히기의 마지막 단계 (불가능한 경우)**

```ts
type A = string | number;

function assertNever(x: A) {
  if (typeof x === 'string') {
    console.log('string');
  } else if (typeof x === 'number') {
    console.log('number');
  } else {
    // 여기는 절대 도달할 수 없음
    const _: never = x; // 에러 발생 가능
  }
}
```

→ 타입스크립트는 여기서 `x`가 `never`여야만 안전하다고 판단합니다.
이 패턴은 **유니언 타입이 빠짐없이 처리되었는지 확인할 때** 자주 사용돼요.

---

## ✅ 실무에서 자주 쓰이는 상황

### 🔹 1. `switch` 문 exhaustive 체크

```ts
type Shape = { kind: 'circle' } | { kind: 'square' };

function draw(shape: Shape) {
  switch (shape.kind) {
    case 'circle':
      // ...
      break;
    case 'square':
      // ...
      break;
    default:
      const _exhaustive: never = shape;
    // 여기서 새로운 kind가 생기면 컴파일 에러 발생!
  }
}
```

> → **추후 타입이 바뀌었을 때 실수 방지**할 수 있는 강력한 도구입니다.

---

### 🔹 2. 잘못된 조건 분기에서 `never` 발생

```ts
function test(x: never) {
  // x는 어떤 값도 들어올 수 없음
}
```

이런 함수에 `string` 같은 걸 넘기면 컴파일 에러가 나요.
→ "이 코드가 절대 실행되면 안 된다"는 걸 명시적으로 표현하는 수단이에요.

---

## 📌 정리

| 특징        | 설명                                    |
| ----------- | --------------------------------------- |
| 값 없음     | 어떤 값도 할당될 수 없음                |
| 함수에서    | 절대 반환되지 않는 경우에 사용          |
| 타입 가드   | 잘못된 분기 처리 확인에 사용            |
| 안전성 확보 | 모든 유니언을 처리했는지 확인할 때 유용 |

---

## 🧶 마무리

`never`는 **타입 시스템에서 오류나 불가능한 상황을 명확하게 표현**하는 도구입니다.
런타임이 아니라 **컴파일 타임에서 코드 안정성을 확보**하는 데 매우 중요한 역할을 해요.

> 3년 차 개발자라면 `never`을 마주쳤을 때 "이건 의도된 불가능한 경우구나" 하고 바로 알아차릴 수 있어야 합니다.

---

필요하다면 `never` 관련한 실전 사례나, `infer`와 함께 쓰는 고급 패턴도 추가로 소개해드릴게요.

## as const?

좋아요!
이번에는 아래 코드를 기반으로 **`as const`, `typeof ...[number]`, 그리고 리터럴 타입이 무엇인지**까지 포함해서 한 번에 정리해드릴게요.

---

# 🔍 코드

```ts
const eventTypes = ['click', 'keydown', 'focus'] as const;
type EventType = (typeof eventTypes)[number];
```

이건 **값을 기반으로 타입을 만드는 고급 타입스크립트 패턴**입니다.
이걸 제대로 이해하려면 `리터럴 타입`, `as const`, `typeof`, `number 인덱스` 이 네 가지 키워드를 이해해야 해요.

---

## 📌 1. 리터럴 타입(Literal Type)이란?

> **리터럴 타입**은 `'hello'`, `42`, `true` 처럼 **특정한 값 하나만을 나타내는 타입**입니다.

예:

```ts
let a: 'click'; // a에는 'click'만 들어올 수 있음
let b: 42; // b에는 42만 들어올 수 있음
```

즉, `"문자열" | 숫자 | boolean"`을 다 포함하는 범용 타입이 아니라,
**딱 한 가지 값**만 허용하는 아주 좁은 타입을 뜻합니다.

---

## ✅ 2. `as const`란?

```ts
const eventTypes = ['click', 'keydown', 'focus'] as const;
```

기본적으로 배열을 선언하면 타입스크립트는 아래처럼 추론합니다:

```ts
const eventTypes = ['click', 'keydown', 'focus'];
// 타입: string[]
```

하지만 이건 `"click" | "keydown" | "focus"` 중 무엇인지 구분이 안 됩니다.

→ **`as const`를 붙이면**:

```ts
// 타입: readonly ['click', 'keydown', 'focus']
```

- 각 요소가 `'click'`, `'keydown'`, `'focus'`라는 **리터럴 타입**이 되고
- 배열도 `readonly`로 바뀌어 수정할 수 없게 됩니다

> 즉, **값을 타입으로 승격**시켜주는 역할을 해요.

---

## ✅ 3. `typeof eventTypes[number]`란?

```ts
type EventType = (typeof eventTypes)[number];
```

- `typeof eventTypes`: 변수 `eventTypes`의 **타입을 가져오라**는 뜻 → `readonly ['click', 'keydown', 'focus']`
- `eventTypes[number]`: 배열의 인덱스를 통해 접근할 수 있는 모든 타입의 **합집합 (유니언)**

즉,

```ts
type EventType = 'click' | 'keydown' | 'focus';
```

이렇게 유니언 타입을 자동으로 만들어주는 아주 유용한 방식이에요.

---

## 🧠 그래서 이 패턴의 의미는?

```ts
const eventTypes = ['click', 'keydown', 'focus'] as const;
type EventType = (typeof eventTypes)[number];
```

이건 \*\*"eventTypes 배열에 들어있는 값들을 그대로 타입으로 만들자"\*\*는 뜻입니다.
그리고 그 결과 만들어진 `EventType`은 `'click' | 'keydown' | 'focus'`라는 **리터럴 유니언 타입**이 됩니다.

---

## ✅ 실무에서의 활용 예

### 1. 타입 안정성 있는 이벤트 핸들러

```ts
function addEventListener(type: EventType) {
  // type은 'click' | 'keydown' | 'focus' 중 하나만 가능
}

addEventListener('click'); // ✅
addEventListener('dblclick'); // ❌ 컴파일 에러!
```

### 2. API 타입/상수 동기화

```ts
const status = ['idle', 'loading', 'success', 'error'] as const;
type Status = (typeof status)[number];
```

→ 실수로 `"succes"` 같은 오타를 쓸 수 없게 막아줍니다.

---

## 🧶 정리

| 요소                    | 설명                                                                      |
| ----------------------- | ------------------------------------------------------------------------- |
| **리터럴 타입**         | `'click'`, `42`, `true`처럼 "값 자체"를 타입으로 표현                     |
| **as const**            | 배열의 값을 리터럴 타입으로 고정하고, `readonly`로 바꿈                   |
| **typeof ...\[number]** | 배열의 **모든 요소를 유니언 타입으로 추출**                               |
| **활용 예시**           | `EventType`, `Status`, `Role`, `TabKey` 등 제한된 문자열 세트에 자주 사용 |

---

저는 enum을 쓰기 싫어서 이렇게 쓰는데요

```ts
export const CURRENCY_TYPE = {
  USD: 'usd',
  KRW: 'krw',
  EUR: 'eur',
  JPY: 'jpy',
  CNY: 'cny',
} as const;

export const CURRENCY_SYMBOL = {
  [CURRENCY_TYPE.USD]: '$',
  [CURRENCY_TYPE.KRW]: '₩',
  [CURRENCY_TYPE.EUR]: '€',
  [CURRENCY_TYPE.JPY]: '¥',
  [CURRENCY_TYPE.CNY]: '¥',
} as const;

export type CurrencyType = ValueOf<typeof CURRENCY_TYPE>;

// CurrencyType는 이런 리터럴 타입을 갖게됨.
// (alias) type CurrencyType = "usd" | "krw" | "eur" | "jpy" | "cny"
```

이때 CURRENCY_TYPE값에 따른 기호를 표시해줘야한다면
`currency:CurrencyType`
`CURRENCY_SYMBOL[currency]`

이렇게하면 타입안정성도 챙기고 좋다.
