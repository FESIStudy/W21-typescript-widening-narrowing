## 타입스크립트

### 타입스크립트란?

- 타입스크립트 = **자바스크립트의 정적 타입 검사자**
    - 코드가 실행되기 전에 실행하고(정적), 프로그램 타입이 정확한지 확인하는 도구(타입 검사)
- ‘정적 타입 검사자’의 의미
    - 타입스크립트는 개발 중(컴파일 타임)에만 동작하는 타입 검사기일 뿐, 브라우저나 Node.js는 오직 자바스크립트만 실행할 수 있다. **타입스크립트에서의 타입 정보는 런타임에서는 존재하지 않기 때문에, 런타임에서는 타입을 알 수 없다.**
        
        ```tsx
        const msg = "hello" as const;
        
        type MsgType = typeof msg;  // "hello"로 추론됨 (TS의 typeof)
        
        console.log(typeof msg);  // "string" (JS의 typeof)
        ```
        
- 자바스크립트 위에서 **타입 시스템 레이어**의 역할을 해주는 것이 타입스크립트
- 타입스크립트는 다음과 같은 것들을 해준다.
    - 타입 추론
    - 타입 정의
    - 타입 구성 (유니언, 제네릭 등)
    - 구조적 타입 시스템

<br/>

### 타입 추론

- 타입 추론이란 **타입스크립트가 코드를 해석해 나가는 동작**을 의미한다.
- 변수를 선언하거나 초기화할 때, 변수, 속성, 인자의 기본 값, 함수의 반환 값 등을 설정할 때 타입이 추론된다.
    
    ```tsx
    let x = 3;  // x는 number로 간주된다.
    ```
    
- 타입 추론 방식
    1. 최적 공통 타입 (Best Common Type)
        - 보통 몇 개의 표현식을 이용하여 가장 근접한 타입을 추론하게 되는데, 이 가장 근접한 타입을 Best Common Type이라고 한다.
        - 최적 공통 타입 알고리즘은 각 후보 타입을 고려하여 **모든 후보 타입을 포함할 수 있는 타입**을 선택한다.
            
            ```tsx
            let arr = [0, 1, null];  // arr는 (number | null)[] 타입으로 추론된다.
            ```
            
        - 후보 타입들로부터 최적 공통 타입을 선택하기 때문에, 모든 후보 타입을 포함하는 상위 타입이 존재하더라도 후보 타입 중 상위 타입이 존재하지 않으면 선택할 수 없다. 따라서 상위 타입으로 추론되길 원하는 경우 직접 상위 타입을 표기해야 한다.
    2. 문맥상 타이핑 (Contextual Typing)
        - 문맥상의 타입 결정은 코드의 위치(문맥)를 기준으로 일어난다.
            
            ```tsx
            // 오른쪽 함수가 window.onscroll에 할당되었기 때문에 함수의 인자인 uiEvent는 UIEvent로 간주된다.
            // UIEvent에는 button 프로퍼티가 없으므로 console.log에서 에러가 발생한다.
            window.onscroll = function(uiEvent) {
            	console.log(uiEvent.button);  // Error
            }
            
            // 함수 표현식은 위와 동일하지만, 함수가 할당되는 변수명이 달라졌다.
            // 해당 변수명만으로는 타입을 추정하기 어렵기 때문에 아무 에러가 나지 않는다. (uiEvent가 any 타입으로 추론됨)
            const handler = function(uiEvent) {
            	console.log(uiEvent.button);  // Error 발생하지 않음
            }
            ```
            
- 구조적 타입 시스템
    - 타입 체크는 값의 **형태**에 기반하여 이루어진다. (= 두 객체가 같은 형태를 가지면 같은 것으로 간주된다.)
        
        ```tsx
        interface Point {
        	x: number;
        	y: number;
        }
        
        function printPoint(p: Point) {
        	console.log(`${p.x} ${p.y}`);
        }
        
        const point = { x: 12, y: 14 };
        printPoint(point);  // '12 14' 출력
        ```
        
    - 이를 Duck Typing 또는 Structural Subtyping이라고 한다.
        - Duck Typing: 객체의 변수 및 메서드의 집합이 객체의 타입을 결정하는 것
        - Structural Subtyping: 객체의 실제 구조나 정의에 따라 타입을 결정하는 것

<br/>

### 타입 추론의 한계

- 초기 값에 의존한다.
    
    ```tsx
    let x = null;  // x: any
    let y = [];    // y: any[]
    ```
    
    - `null`, `[]`, `undefined` 등으로 초기화하면 구체적인 타입을 추론할 수 없어서 `any`가 된다.
    - 그리고 이후에 값을 넣어도 추론되는 타입은 바뀌지 않는다.
- 복잡한 코드나 동적 타입 변환이 발생하는 경우, 컴파일러가 정확한 타입을 추론하지 못할 수 있다.

<br/>

### Strict 모드와 타입 추론

- Strict 모드는 타입스크립트에서 타입 검사를 더욱 엄격하게 수행하도록 설정하는 옵션이다. `tsconfig.json`에서 `strict: true`를 설정하여 활성화할 수 있다.
- 하위 옵션
    - `strictNullChecks`: `null`과 `undefined`를 다른 타입과 명확히 구분한다. (`null`과 `undefined` 값이 모든 타입의 서브타입으로 속하지 않게 한다.)
        
        ```tsx
        // strictNullChecks: false일 때
        let str: string = null;  // 허용
        let num: number = undefined;  // 허용
        
        // strictNullChecks: true일 때
        let str: string = null;  // 오류
        let num: number = undefined;  // 오류
        
        let str2: string | null = null;  // 허용
        let num2: number | undefined = undefined;  // 허용
        ```
        
    - `noImplicitAny`: 암묵적으로 타입이 `any`로 추론될 경우, 에러를 발생시켜 명시적으로 타입을 지정하게 해준다.
        
        ```tsx
        // Parameter 'x' and 'y' implicitly have an 'any' type. 
        function add(x, y) {
        	return x + y;
        }
        
        // 해결
        function add(x: number, y: number):number {
        	return x + y;
        }
        ```
        
    - `strictFunctionTypes`
        - false → 함수 매개변수가 이변적
        - true → 함수 매개변수가 반공변적
        
        ```tsx
        // strictFunctionTypes: true인 경우
        
        type Animal = { name: string };
        type Dog = { name: string, breed: string };
        
        let dogFunc: (d: Dog) => void;
        let animalFunc: (a: Animal) => void;
        
        dogFunc = animalFunc;  // 에러 발생
        ```
        
    - `strictPropertyInitialization`: 클래스의 프로퍼티는 생성자에서 반드시 초기화되어야 한다. `strictNullChecks` 옵션과 함께 사용해야 한다.
        
        ```tsx
        class User {
        	name: string;  // 오류: name이 초기화되지 않음
        	
        	constructor() {
        		// name을 초기화하지 않으면 컴파일 에러 발생
        	}
        }
        ```
        
    - `noImplicitThis`: `this`의 타입이 명확하지 않으면 오류를 발생시킨다.
    - `alwaysStrict`: 모든 파일에 `use strict`를 자동으로 추가한다.
        - 런타임에서 자바스크립트의 strict mode를 활성화한다. (`use strict` 삽입)

<br/>

## 🤔  공변성과 반공변성, 이변성

- **타입 간의 포함 관계가 유지되는 방향을 의미하는 개념**으로, 특히 제네릭 타입이나 함수 타입의 안전한 **대입** 여부를 판단할 때 중요하게 사용된다.
- 상위 타입과 하위 타입
    - 상위 타입: 더 일반적인 타입
    - 하위 타입: 더 구체적인 타입, 상위 타입이 할 수 있는 모든 것을 할 수 있고, 그 이상도 할 수 있는 타입
    - **할당 가능성**을 기준으로 판단하면 된다.
        
        ```tsx
        // A가 B의 하위 타입인지 확인하는 방법
        // B 타입 변수에 A 타입 값을 할당할 수 있는가?
        
        let a: A = /* 어떤 값 */;
        let b: B = a;  // 이것이 가능하면 A <: B
        ```

<br/>

### 공변성 (Covariance)

- **하위 타입 관계가 그대로 유지되는 성질**
- 배열에서의 공변성
    
    ```tsx
    class Animal {
      name: string = "";
    }
    
    class Dog extends Animal {
      breed: string = "";
    }
    
    // Dog <: Animal 이므로 (A가 B의 하위 타입인 경우 A <: B로 표현)
    // Dog[] <: Animal[] 도 성립 (공변성)
    let dogs: Dog[] = [new Dog()];
    let animals: Animal[] = dogs; // 가능
    ```
    
- 함수 반환 타입에서의 공변성
    
    ```tsx
    type AnimalProvider = () => Animal;
    type DogProvider = () => Dog;
    
    // Dog <: Animal 이므로
    // DogProvider <: AnimalProvider (반환 타입은 공변적)
    let dogProvider: DogProvider = () => new Dog();
    let animalProvider: AnimalProvider = dogProvider; // 안전
    
    let animal = animalProvider(); // Animal 타입으로 받지만 실제로는 Dog
    ```
    
<br/>

### 반공변성 (Contravariance)

- **하위 타입 관계가 뒤바뀌는 성질**
- 함수 매개변수에서의 반공변성
    
    ```tsx
    type AnimalHandler = (animal: Animal) => void;
    type DogHandler = (dog: Dog) => void;
    
    // Dog <: Animal 이지만
    // AnimalHandler <: DogHandler (매개변수는 반공변적)
    
    let animalHandler: AnimalHandler = (animal) => {
      console.log(animal.name); // Animal의 속성만 사용
    };
    
    let dogHandler: DogHandler = animalHandler; // 안전
    
    // 실제로 실행되는 것은 animalHandler(new Dog())와 동일
    // Dog는 Animal을 상속하고 있으므로
    // Dog를 전달해도 animalHandler가 Animal로 처리 가능
    dogHandler(new Dog()); 
    ```
    
- 왜 반공변적인가?
    
    ```tsx
    // 위험한 상황을 가정해보면
    let unsafeDogHandler: DogHandler = (dog) => {
      console.log(dog.breed); // Dog 특정 속성 사용
    };
    
    // 만약 공변적이라면 이것이 가능해야 하는데...
    // let animalHandler: AnimalHandler = unsafeDogHandler; // 위험
    ```
   
<br/> 

### 공변성과 반공변성 비유로 이해하기

- 공변성
    - 구체적인 것을 일반적인 것에 넣어줄 수 있다.
    - **주는 쪽(생산자)**에서 안전
    - ex) 과일 먹는 사람에게 사과 주기
- 반공변성
    - 일반적인 것은 구체적인 것을 받을 수 있다.
    - **받는 쪽(소비자)**에서 안전
    - ex) 음식 먹는 사람이 사과 먹기

<br/>

### 이변성 (Bivariance)

- 공변성과 반공변성을 모두 허용하는 성질
- 메서드 매개변수의 이변성
    
    ```tsx
    class AnimalTrainer {
      train(animal: Animal): void {
        console.log(`Training ${animal.name}`);
      }
    }
    
    class DogTrainer {
      train(dog: Dog): void {
        console.log(`Training dog ${dog.breed}`);
      }
    }
    
    // TypeScript는 기본적으로 메서드 매개변수에 대해 이변적
    let animalTrainer: AnimalTrainer = new AnimalTrainer();
    let dogTrainer: DogTrainer = new DogTrainer();
    
    // 양방향 할당이 모두 가능 (이변성)
    animalTrainer = dogTrainer; // ✅ 허용
    dogTrainer = animalTrainer; // ✅ 허용 (하지만 위험할 수 있음)
    ```
    
<br/>

### 불변성 (Invariance)

- 정확히 같은 타입만 허용하는 성질
- 제네릭 타입의 불변성
    
    ```tsx
    // 제네릭 인터페이스
    interface Box<T> {
      value: T;
      setValue(value: T): void;
      getValue(): T;
    }
    
    let dogBox: Box<Dog> = {
      value: new Dog(),
      setValue(dog: Dog) { this.value = dog; },
      getValue() { return this.value; }
    };
    
    let animalBox: Box<Animal>;
    
    // 불변적 관계 - 양방향 모두 불가능
    // animalBox = dogBox; // 에러
    // dogBox = animalBox; // 에러
    ```
    
<br/>

## 타입 넓히기 (Type Widening)

- 변수를 초기화할 때 타입을 명시하지 않으면 타입 체커는 타입을 결정해야 한다. 이때 타입스크립트는 **지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추**하는데, 이 과정을 타입 넓히기라고 한다.
    - 리터럴 타입이 일반적인 타입으로 확장되는 것
    - 타입 추론의 일부로, 불필요하게 엄격한 리터럴 타입으로 인해 코드 사용성이 떨어지는 것을 방지하기 위해 도입된 동작이다.
- 예시
    
    ```tsx
    // 문자열 리터럴 → string (선언 시점에 넓히기 발생)
    let greeting = 'hello';  // greeting: string 
    ```
    
    ```tsx
    // 숫자 리터럴 → number
    let age = 20;  // age: number
    ```
    
    ```tsx
    // undefined/null -> any
    let v = undefined;  // v: any (타입을 추론할 수 없어 any로 넓혀짐)
    let x = null;  // x: any
    ```
    
    ```tsx
    // 객체 리터럴
    let user = { name: "Tom" };  // user: { name: string }
    ```
    
    ```tsx
    // 배열
    const arr = [1, 2, 3]; // number[]
    const arr2 = [1, "hello"]; // (string | number)[]
    const arr3 = [] as const; // readonly []
    ```
    
    ```tsx
    // 함수 반환 타입
    function getValue() {
        return "hello"; // 반환 타입이 string으로 넓혀짐
    }
    ```
    
- 타입 넓히기가 진행될 때는 주어진 값으로 추론 가능한 타입이 여러 개이므로 그 과정이 모호하다. 따라서 타입 넓히기로 인해 문제가 발생하기도 한다.
    
    ```tsx
    const makeRequest = (method: "GET" | "POST") => {};
    
    let method = "GET";
    makeRequest(method);  // method가 string으로 추론되어 오류 발생
    ```
    
<br/>

### 타입 넓히기 제어하기

따라서 타입 넓히기 과정을 제어하기 위해 다음과 같은 방법을 사용할 수 있다.

- `let` 대신 `const` 사용
    - `const`를 사용해 변수를 선언할 경우, 선언과 동시에 값이 할당되고 재할당이 불가능하므로 타입스크립트는 좁은 타입으로 추론할 수 있다.
        
        ```tsx
        const greeting = "hello";  // greeting: "hello"
        ```
        
- 타입 체커에게 추가적인 문맥 제공
    - `const`를 사용해도 객체와 배열에서는 문제가 발생할 수 있다.
    - 예시) 객체의 경우 타입스크립트의 넓히기 알고리즘은 각 요소를 let으로 할당된 것처럼 다룬다.
        
        ```tsx
        // v: { x: number }로 추론됨 → 값이 1이어도 number로 widen됨
        const v = {
        	x: 1,
        }
        
        v.x = 2;
        v.x = '2';  // 오류: '2'는 number 형식에 할당할 수 없음
        v.y = 3;  // 오류: { x: number }에 'y' 속성이 없음
        ```
        
        ```tsx
        // 함수 매개변수를 활용해 문맥적 타입 제공
        function doSomething(p: { x: 1 }) {
        	// 타입 체커는 p.x가 number가 아닌 정확히 1임을 인지
        }
        
        doSomething({ x: 1 });  // OK. x는 리터럴 1로 추론됨
        ```
        
- 명시적 타입 선언
    
    ```tsx
    let method = "GET" | "POST" = "GET";
    ```
    
- 타입 단언
    
    ```tsx
    let status = "success" as "success";
    ```
    
- `as const` 사용
    
    ```tsx
    const status = "success" as const;  // status: "success" (string으로 타입이 넓혀지지 않음)
    
    const obj = {
    	type: "post",     // obj.type: "post"
    	published: true   // obj.published: true
    } as const;
    ```
    
<br/>

## 타입 좁히기 (Type Narrowing)

- 타입 넓히기와 반대로, 변수나 표현식의 타입을 추론할 때 **더 좁은 범위의 타입으로 추론하는 것**을 의미한다. (넓은 타입을 더 좁은 타입으로 재정의하는 행위)
- **타입 가드** (Type Guard)
    - 특정 스코프 내에서 타입 좁히기를 유발하는 표현을 타입 가드라고 부르며, 크게 두 종류로 나뉜다.
        - **제어 흐름 분석**(control flow analysis)을 통해 타입을 좁히는 가드
        - 프로그래머가 값을 어떤 타입으로 좁혀야 하는지 명시할 수 있는 **사용자 정의 타입 가드**(user defined type guard)

### 제어 흐름 분석을 통한 타입 좁히기

> 대부분의 프로그래밍 언어는 제어 구조를 가진다. (`if`, `for`, `break`, `continue`, `return` 등을 제공)
컴파일러는 이런 제어 구조로부터 코드의 특정 시점에서 프로그램이 갖는 상태에 대한 정보를 얻어낼 수 있고, 이러한 정보를 이용해 제어 흐름 분석을 진행해 특정 값의 타입을 좁힐 수 있다.
> 
- `null`/`undefined` 과의 비교
    
    ```tsx
    function greet(name: string | null) {
    	if (name) {
    		console.log("Hello " + name.toUpperCase());  // name: string
    	} else {
    		console.log("Hello stranger");  // name: null
    	}
    }
    ```
    
- `typeof` 연산자 사용
    - 주의) `typeof null`은 ‘object’를 반환하므로 null 체크는 `===` 연산자 사용
    
    ```tsx
    function print(value: string | number) {
    	if (typeof value === "string") {
    		// 여기서 value는 string
    		console.log(value.toUpperCase());
    	} else {
    		// 여기서 value는 number
    		console.log(value.toFixed(2));
    	}
    }
    ```
    
- `in` 연산자 사용 (객체 속성 체크)
    
    ```tsx
    type Dog = { bark: () => void };
    type Cat = { meow: () => void };
    
    function makeSound(animal: Dog | Cat) {
    	if ("bark" in animal) {
    		animal.bark();  // animal은 Dog
    	} else {
    		animal.meow();  // animal은 Cat
    	}
    }
    ```
    
- `instanceof` 연산자 사용 (클래스 타입 좁히기)
    - `instanceof` 연산자는 값과 생성자를 받아 해당 값의 프로토타입 체인에 해당 생성자가 등장하는지를 확인한다.
    
    ```tsx
    class A {
    	a = 1;
    }
    
    class B {
    	b = 2;
    }
    
    function process(x: A | B) {
    	if (x instanceof A) {
    		console.log(x.a);  // 1
    	} else {
    		console.log(x.b);  // 2
    	}
    }
    ```
    
- Discriminated Union 패턴
    - 공통된 `tag` 또는 `kind` 필드를 기준으로 타입을 구분하는 방식
    - 복잡한 union 타입을 다룰 때 가장 안정적이고 직관적인 타입 좁히기 방식이다.
    
    ```tsx
    type Shape = 
    	| { kind: "circle", radius: number }
    	| { kind: "square", side: number };
    	
    function getArea(shape: Shape) {
    	if (shape.kind === "circle") {
    		return Math.PI * shape.radius ** 2;
    	} else {
    		return shape.side ** 2;
    	}
    }
    ```
    
- Exhaustiveness check (`never` 타입을 이용한 소진 검사)
    - 모든 union case를 빠짐없이 처리했는지 검사하는 방식으로, 빠진 경우에 `never`로 에러가 발생하도록 한다.
    
    ```tsx
    function handleShape(shape: Shape) {
    	switch (shape.kind) {
    		case "circle":
    			return shape.radius;
    		case "square":
    			return shape.side;
    		default:
    			const _exhaustiveCheck: never = shape;
    			// 만약 새로운 kind가 추가되었는데 case가 빠지면 여기에서 에러가 발생한다.
    		}
    	}
    ```
    
<br/>

### 사용자 정의 타입 가드

- 사용자 정의 타입 가드는 `value is Type` 형태의 반환 타입을 가지는 **함수**로 정의한다.
- `is` 키워드: 함수가 특정 타입을 판별한다는 것을 타입 체커에게 알려주는 역할을 한다.
    
    ```tsx
    type Fish = { swim: () => void };
    type Bird = { fly: () => void };
    
    // 함수가 true를 반환하면, animal은 Fish 타입이라고 타입스크립트에게 알려준다.
    function isFish(animal: Fish | Bird): animal is Fish {
      return 'swim' in animal;
    }
    
    function move(animal: Fish | Bird) {
      if (isFish(animal)) {
        animal.swim(); // Fish
      } else {
        animal.fly();  // Bird
      }
    }
    ```
    
- 타입 가드 실습
    
    ```tsx
    // API 요청 시 에러가 발생한 경우, 서버에서 다음과 같은 타입의 객체를 응답으로 보내주는 상황
    export interface ErrorResponse {
    	status: 'failure' | 'error';
    	data: null;
    	message: string;
    }
    
    // 런타임에 ErrorResponse 타입인지 검사하는 타입 가드 함수
    export const isErrorResponse = (data: unknown): data is ErrorResponse =>
      data !== null &&
      typeof data === 'object' &&
      'status' in data &&
      (data.status === 'failure' || data.status === 'error') &&
      'message' in data &&
      typeof data.message === 'string' &&
      'data' in data &&
      data.data === null;
      
      
    // 실제 사용 코드
    api.interceptors.response.use(
      (response) => response, 
      (error: AxiosError) => {
        if (error.response) {
          const { data } = error.response;
    
          if (isErrorResponse(data)) {
            // 서버에서 내려준 CustomError 타입인 경우
            return Promise.reject(
              new CustomAxiosError(error, 'business error', data.message),
            );
          }
    
          // 서버가 응답을 보냈으나, CustomError 타입이 아닌 경우
          return Promise.reject(
            new CustomAxiosError(
              error,
              'unexpected error',
              '예상치 못한 오류가 발생했습니다. 관리자에게 문의해 주세요.',
            ),
          );
        }
        
        // ... 생략
      },
    );
    ```
    

> 초기 버전의 타입스크립트는 타입 시스템의 힘이 강력하지 않았고, 제어 흐름 분석에 기반한 타입 좁히기가 거의 이루어지지 않았다. 때문에 사용자 정의 타입 가드를 사용해야 하는 경우가 많았다. 하지만 꾸준히 발전을 거듭한 오늘날의 타입스크립트에선 타입 좁히기가 똑똑하게 이루어지고, 대부분의 사용례는 위에서 다룬 내장 타입 가드로도 충분히 커버할 수 있다.
(출처: https://ahnheejong.gitbook.io/ts-for-jsdev/06-type-system-deepdive/type-narrowing)
> 

<br/>

## 복잡한 타입 연산자와 타입 추론

### 조건부 타입 (`T extends string ? A : B`)

- 조건부 타입은 타입 레벨의 if문으로, 제네릭 타입 `T`가 어떤 조건을 만족하는지에 따라 타입을 분기시킨다.
- 타입 추론 흐름을 가지치기하는 데 자주 사용된다.

```tsx
type ToString<T> = T extends string ? string : never;

type A = ToString<"hello">;  // string
type B = ToString<number>;   // never
```

<br/>

### 제네릭 사용 시 widening/narrowing 컨트롤

- 제네릭을 사용할 때는 인자로 전달되는 타입이 넓혀질 수도 있고, 좁혀질 수도 있다.
- 특히 객체 리터럴은 컨텍스트가 없으면 자동으로 타입이 넓혀진다.
    
    ```tsx
    function wrap<T>(value: T): T {
    	return value;
    }
    
    const a = wrap("hello");  // T: string (타입 넓혀짐)
    const b = wrap({ x: 1 });  // T: { x: number } (x 또한 타입이 넓어짐)
    ```
    
- `as const`나 명시적 타입 지정을 통해 제네릭의 widening을 제어할 수 있다.
    
    ```tsx
    const c = wrap({ x: 1 } as const);  // T: { x: 1 }
    ```
    
<br/>

### `keyof`, mapped types에서의 타입 흐름

- `keyof`: 제공된 타입의 키를 추출하여 새로운 Union 유형으로 반환한다.
- mapped types: 다른 타입을 바탕으로 새로운 타입을 생성한다.

```tsx
type User = { name: string; age: number };
type Keys = keyof User;  // "name" | "age"

type PartialUser = {                  // { name?: string; age?: number }
	[K in keyof User]?: User[K];
};
```

- 이때 타입스크립트는 각 K마다 타입 흐름을 추적하면서 필요한 경우 widening/narrowing을 수행한다.
- mapped type에서도 `as const`나 리터럴 타입 유지를 통해 정확한 key 타입을 보존할 수 있다.


<br/>

### `ReturnType<T>`와 widening

- `ReturnType<T>`: 함수의 반환 타입을 추론하는 유틸리티 타입이다.
- 그러나 함수 리터럴을 바로 넣으면 타입스크립트가 widen된 타입을 추론할 수 있다.
    
    ```tsx
    const getUser = () => ({
    	name: "Alice",
    	age: 30,
    });
    
    type User = ReturnType<typeof getUser>;  // { name: string; age: number }
    ```
    
- 이 경우에도 정확한 리터럴 타입을 유지하고 싶다면 `as const`를 사용하거나, 함수의 반환 타입을 명시해주면 된다.
    
    ```tsx
    const getUser = () => ({
    	name: "Alice",
    	age: 30,
    } as const);
    
    type LiteralUser = ReturnType<typeof getUser>;  // { readonly name: "Alice"; readonly age: 30 }
    
    // 또는
    const getUser = (): { name: "Alice", age: 30 } => ({
    	name: "Alice",
    	age: 30,
    });
    
    type LiteralUser = ReturnType<typeof getUser>;  // { name: "Alice"; age: 30 }
    ```
    
    - 🤔 `as const`를 사용하는 것과 반환 타입을 명시하는 것은 뭐가 다를까?
        - 차이점 1) `readonly` 속성
            - `as const`는 리터럴 타입 + `readonly` 속성을 자동으로 적용하지만, 명시적 타입은 리터럴 타입만 보존할 뿐, `readonly`는 명시하지 않으면 자동으로 적용되지 않는다.
        - 차이점 2) 타입 오류 발생 시점
            
            
            | 방식 | 동작 방식 | 타입 검사 흐름 | 오류 발생 시점 |
            | --- | --- | --- | --- |
            | `as const` | 값을 먼저 보고 타입을 추론 | 값 → 타입 추론 → 타입 적용 | 추론된 타입 사용 시점 |
            | 명시적 타입 | 타입을 먼저 보고, 값이 그 타입과 맞는지만 검사 | 타입 → 값 검사 | 값이 타입과 불일치할 때 바로 |

<br/>

### 🤔 `as const`와 `readonly`의 차이

- `readonly`
    - **객체나 배열의 속성을 수정할 수 없게 만드는** 타입 제어 키워드
    
    ```tsx
    type User = {
    	readonly name: string;
    };
    
    const user: User = { name: "Alice" };
    
    user.name = "Bob";  // 'name'은 읽기 전용이므로 오류 발생
    ```
    
- `as const`
    - 값 전체를 **리터럴+불변**으로 만드는 타입 단언이다.
    - **타입을 리터럴 타입으로 좁히고**, 모든 속성에 `readonly`를 자동으로 추가한다.
    
    ```tsx
    const user = {        // user: { readonly name: "Alice"; readonly age: 30 }
    	name: "Alice",
    	age: 30,
    } as const;
    ```
    
- `readonly`는 타입에서, `as const`는 값에서 쓰며 리터럴 유지 여부에 차이가 있다.

<br/>

## satisfies 연산자

- 타입 안전성을 보장하면서도 변수의 추론된 타입을 보존할 수 있게 해주는 연산자이다.
    
    ```tsx
    const value = expression satisfies Type;
    ```
    
    - `satisfies` 연산자로 `Type` 타입을 지정하면, 해당 변수의 실제 항목을 미리 `Type` 타입에 대입해 보면서 체크 및 밸리데이팅을 수행해준다.
        
        ```tsx
        type Colors = "red" | "green" | "blue";
        
        // 내부적으로 이런 검증 과정을 거침
        const palette = {
          primary: "red",     // ✅ "red"가 Colors에 할당 가능한가? → Yes
          secondary: "yellow" // ❌ "yellow"가 Colors에 할당 가능한가? → No, 컴파일 에러
        } satisfies Record<string, Colors>;
        ```
        
<br/>

### `satisfies` vs `as` vs 타입 선언

- 타입 선언과의 차이점
    
    ```tsx
    // 타입 선언 사용 시
    const config: Record<string, string | number> = {
    	apiUrl: 'https://api.example.com',
    	timeout: 5000,
    	retries: 3
    };
    
    // 문제: 타입스크립트가 더 넓은 타입으로만 인식한다.
    config.apiUrl.toUpperCase();  // 에러: string | number에는 toUpperCase가 없음
    ```
    
    ```tsx
    // satisfies 사용 시
    const configSatisfies = {
    	apiUrl: 'https://api.example.com',
    	timeout: 5000,
    	retries: 3
    } satisfies Record<string, string | number>;
    
    configSatisfies.apiUrl.toUpperCase();  // string으로 추론되어 사용 가능
    ```
    
- `as` (타입 단언)과의 차이점
    1. 타입 검증 여부
        
        ```tsx
        type Colors = "red" | "green" | "blue";
        
        // as: 타입 검증 없음 (강제 변환)
        const color1 = "yellow" as Colors;         // 런타임 에러 가능성, 컴파일은 통과
        
        // satisfies: 타입 검증 있음
        const color2 = "yellow" satisfies Colors;  // 컴파일 에러로 미리 잡아냄
        const color3 = "red" satisfies Colors;     // 정상
        ```
        
    2. 타입 정보 처리 방식
        
        ```tsx
        type APIResponse = {
          data: unknown;
          status: number;
        };
        
        const response1 = {
          data: { name: "John", age: 30 },
          status: 200
        } as APIResponse;
        
        const response2 = {
          data: { name: "John", age: 30 },
          status: 200
        } satisfies APIResponse;
        
        // as 사용 시: 타입이 APIResponse로 고정됨
        response1.data.name;  // 에러 발생 data는 unknown 타입
        
        // satisfies 사용 시: 구체적인 타입 정보 유지
        response2.data.name;  // data는 { name: string, age: number }로 추론
        response2.status.toFixed(0);  // status는 number로 추론
        ```
        
<br/>

### `satisfies` vs `as const`

- `as const`
    - 목적: 리터럴 타입 고정 + 불변성 보장
    - 효과: 타입 넓히기 방지, readonly 추가, 배열 → 튜플 변환
    - 상수 값이나 설정 객체를 정의할 때 사용
- `satisfies`
    - 목적: 타입 검증 + 추론된 타입 보존
    - 효과: 컴파일 타임 타입 체크, 구체적 타입 정보 유지
    - 타입 안전성을 보장하면서도 정확한 타입 추론을 원할 때
- 서로 다른 목적을 가지고 있기 때문에 함께 사용할 수도 있다.
    - 실무에서는 특히 설정 객체, 상수 정의, API 스키마 검증 등에서 많이 활용된다.
    
    ```tsx
    const config = {
      theme: "dark",
      apiUrl: "https://api.com"
    } as const satisfies AppConfig;
    ```
    
<br/>

## 면접 예상 질문

- Q1. widening은 왜 문제를 일으킬 수 있는가?
    - 타입스크립트의 타입 넓히기(widening)는 초기 변수 선언 시, 리터럴 값이 보다 일반적인 타입으로 자동 변환되는 과정입니다. 타입 넓히기로 인해 개발자의 의도보다 넓은 타입으로 추론이 된다면, 잘못된 값이 할당될 여지가 생기고 런타임 오류의 원인이 됩니다.
- Q2. narrowing이 실패하는 경우는 어떤 케이스가 있는가?
    - 타입 좁히기는 조건문 등을 통해 타입 범위를 줄이는 기법입니다. 타입 가드를 잘못 구현한 경우, 런타임 구조와 타입 구조가 불일치하는 경우, 공통 속성이 없는 경우 등에서 실패할 수 있습니다.
- Q3. discriminated union과 타입 가드의 관계를 설명하라
    - Discriminated Union은 공통의 리터럴 타입 프로퍼티(discriminant)를 기준으로 분기할 수 있는 구조입니다. 이 구조는 자체적으로 discriminant 프로퍼티가 타입 가드 역할을 수행하므로, 안전하게 타입 좁히기가 가능합니다.
- Q4. `as const`와 `satisfies`의 차이점은 무엇이며 어떤 상황에 쓰는가?
    - `as const`와 `satisfies`는 사용하는 목적이 다릅니다.
    - `as const`는 변수나 객체를 선언할 때 그 값을 리터럴 타입으로 고정하고, 내부 속성들을 모두 `readonly`로 만들어주며, 주로 상수 값이나 설정 객체를 정의할 때 사용합니다.
    - 반면 `satisfies`는 값이 어떤 타입을 만족하는지를 컴파일 타임에 검사하는 용도로 사용합니다. 특히 타입을 강제로 덮어씌우지 않고, 타입 검사를 하면서 원래 추론된 리터럴 타입 정보를 그대로 유지한다는 점이 특징입니다.
- Q5. 왜 `as const`는 readonly를 자동으로 붙이나요?
    - `as const`는 타입스크립트에게 “이 값은 완전히 고정된 상수(literal)로 간주하라”는 의미이므로, 속성도 변경되지 않아야 하기 때문에 `readonly`가 자동으로 부여됩니다.

<br/>

### 참고자료

https://typescript-kr.github.io/pages/tutorials/ts-for-js-programmers.html

https://joshua1988.github.io/ts/guide/type-inference.html#%ED%83%80%EC%9E%85-%EC%B6%94%EB%A1%A0-type-inference

https://velog.io/@bsboys/strict-%EB%AA%A8%EB%93%9C

https://medium.com/@yujso66/%EB%B2%88%EC%97%AD-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%9D%98-%EA%B3%B5%EB%B3%80%EC%84%B1%EA%B3%BC-%EB%B0%98%EA%B3%B5%EB%B3%80%EC%84%B1-82139f7e5cc3

https://www.zerocho.com/category/TypeScript/post/5faa8c657753bd00048a27d8

https://velog.io/@ssulv3030/Effective-Typescript-%ED%83%80%EC%9E%85-%EC%B6%94%EB%A1%A02-%ED%83%80%EC%9E%85-%EB%84%93%ED%9E%88%EA%B8%B0-%ED%83%80%EC%9E%85-%EC%A2%81%ED%9E%88%EA%B8%B0

https://ahnheejong.gitbook.io/ts-for-jsdev/06-type-system-deepdive/type-narrowing

https://typescript-exercises.github.io/#exercise=1&file=%2Findex.ts

https://mycodings.fly.dev/blog/2023-07-14-understanding-typescript-satisfies-operator