# typescript 기본 정리

## typescript의 타입 추론 이해하기

1. 상수 초기화 타입 추론

   ```javascript
   // const 초기화 값으로 타입을 추론하기 때문에 불필요한 타입 선언이다.
   const word: string = "name"; // (X)
   const word = "name"; // (O)

   // 단, 의도치 않은 타입인 경우 명시 필요
   const arr = [1, 2, "hi"]; // (X) arr:(string|number)[]
   const arr: [number, number, string] = [1, 2, "hi"]; // (O)
   ```

2. 함수 반환값 타입 추론
   - 매개변수는 타입 선언 해줘야한다.
   ```javascript
   // 함수 반환 값을 타입스크립트가 추론하기 때문에 선언할 필요 없음
   function add(x: number, y: number) {
     return x + y;
   }
   add(1, 2);
   ```

---

## type 과 interface

- interface는 중복 선언이 가능하고, 확장될 수 있다.
  - 서드파티 타입 수정도 가능
- 확장 방식의 차이
  - 뭘 써도 되지만, 중요한 건 일관성을 지키는 것.
  - 의미적으로 extends를 쓰는게 확장을 파악하기 쉬움

```javascript
type Animal = { breath: true };
type Human = Animal & { think: true };
const me: Human = { breath: true, think: true };

interface Animal {
  breath: true;
}
interface Human extends Animal {
  think: true;
}
const me: Human = { breath: true, think: true };
```

---

## 좁은 타입과 넓은 타입

- ex. (string | number) > (string) > (string & number)
- 구체적일수록 좁은 타입
- 좁은 타입은 넓은 타입에 대입할 수 있다.

#### 초과 프로퍼티 검사

- 객체 리터럴을 변수에 직접 할당할 때나 인수로 전달할 때, 초과 프로퍼티 검사(excess property checking)를 받게 된다.
  그래서 만약 객체 리터럴이 "대상 타입(targe type)"이 갖고 있지 않은 프로퍼티를 갖고 있으면,
  타입 에러를 발생시킨다.

```javascript
interface Avengers {
  name: string;
}
let hero: Avengers;
hero = { name: "captain", location: "seoul" }; // '{name: string; location:string;}' 형식은 Avengers 형식에 할당할 수 없습니다.
```

- 객체를 다른 변수에 할당하게 되면, 초과 프로퍼티 검사를 받지 않기 때문에 타입 에러가 없다.
- 추가로, 타입 단언이나 인덱스 시그니처를 사용할 수도 있다.

```javascript
interface Avengers {
  name: string;
}
let hero: Avengers;
let temp = { name: "captain", location: "seoul" };
hero = temp;

// or
let hero: Avengers;
temp = { name: "captain", location: "seoul" } as Avengers;

// or
interface Avengers{
  name: string;
  [propName: string]: any;
}
```

---

## typeof / keyof

- keyof
  - type의 key만 가져오고 싶을 때 사용
- typeof
  - 객체의 type만 가져오고 싶을 때 사용

```javascript
const obj = {
  a: '123',
  b: 'hello'
} as const;

typeof obj // const obj = { a: string, b: string }
type Key = keyof typeof obj // type Key = 'a' | 'b'
type Value = typeof obj[keyof typeof obj]; // type Value = '123' | 'hello'
```

---

## as

- 타입을 강제로 변경시켜줌

```javascript
// obj : { up :number, down: number, ... }
const obj = {
  up: 0,
  down: 1,
  left: 2,
  right: 3
}

// obj : { readonly up: 0, readonly down: 1, ... }
const obj = {
  up : 0,
  down, 1,
  ...
} as const
```

---

## void 타입

- void는 함수의 반환 값이 없을 때(undefined) 사용되는 타입
- 단, void가 매개변수 내에서 쓰이거나, 메서드로 쓰일 때는 타입 에러가 없다.

```javascript
// 반환 값이 없거나 undefined이어야한다.
function fn(): void{
  ...
}

// callback에서 쓰일 때는 값이 있을 수 있다.
function fn(callback: () => void):void{
  ...
}
fn(()=>{ return 3; });

// 메서드로 쓰일 때는 값이 있을 수 있다.
interface Act{
  talk: () => void
}
const obj: Act = {
  talk() { return 10; }
}
```

```javascript
let target: number[] = [];

declare function fn(arr: number[], callback: (el: number) => void): void;
forEach([1, 2, 3], (el) => {
  target.push(el);
}); // 통과
forEach([1, 2, 3], (el) => target.push(el)); // 통과  (target.push(el)는 number 타입 반환)

declare function fn(arr: number[], callback: (el: number) => undefined): void;
forEach([1, 2, 3], (el) => {
  target.push(el);
}); // 타입 에러  : void는 undefined에 할당 불가
forEach([1, 2, 3], (el) => target.push(el)); // 타입 에러  : number는 undefined에 할당 불가
```

---

## any와 unknown

- any : 어떤 타입이 와도 타입 체크가 통과함
  - any는 타입스크립트를 사용하는 의미가 없어지니 최대한 지양
  ```javascript
  try{
    ...
  }catch(error: any){
    console.log(error.messgae);  // 통과
  }
  ```
- unknown : 어떤 타입이 올 지 모르니, 구체적인 타입 체크가 필요함

  - unknown은 사용하는 시점에 as를 통해 타입을 결정하여 사용함
  - error와 같이 어떤 타입이 올 지 모르는 경우 유용함

  ```javascript
  try{
    ...
  }catch(error: unknown){
    console.log(error.messgae);  // 타입 에러

    console.log((error as Error).message) // 통과
  }
  ```

---

## 타입 가드

1. 타입스크립트는 if문으로 타입 구분을 해준다.

```javascript
function numOrStr(a: number | string) {
  if (typeof a === "number") {
    a.toFixed(1); // typescriptrk a의 type을 number로 인식
  } else {
    a.charAt(3); // typescript가 a의 type을 string으로 인식
  }

  if (typeof a === "boolean") {
    a.toString(); // 타입에러 : typescript가 a의 type을 never로 인식
  }
}
```

2. 클래스는 클래스의 이름을 갖는 인스턴스가 타입이 된다.

```javascript
class A {
  aaa() {}
}

function aOrB(param: A | B) {
  if (param instanceof A) {
    param.aaa(); // param의 type은 A이다.
  }
}

aOrB(new A());
```

3. 객체의 속성으로도 타입을 구분할 수 있다.

```javascript
type B = { type: "b", bbb: string };
type C = { type: "c", ccc: string };

function typeCheck(a: B | C) {
  if (a.type === "b") {
    a.bbb; // a는 타입이 B로 인식
  } else if (a.type === "c") {
    a.ccc; // a는 타입이 C로 인식
  } else {
    a.ddd; // 타입에러 : a는 타입이 never로 인식
  }
}

function typeCheck2(a: B | C) {
  if ("bbb" in a) {
    a.bbb; // a는 타입이 B로 인식
  } else if ("ccc" in a) {
    a.ccc; // a는 타입이 C로 인식
  } else {
    a.ddd; // 타입에러 : a는 타입이 never로 인식
  }
}
```

### is 키워드, 커스텀 타입 가드 함수

is 키워드를 통해 타입 가드를 직접 커스텀할 수 있다.

```typescript
interface Cat {
  meow: number;
}
interface Dog {
  bow: number;
}
function catOrDog(a: Cat | Dog): a is Dog {
  if ((a as Cat).meow) return false;
  return true;
}

function typeCheck(a: Cat | Dog) {
  if (catOrDog(a)) {
    console.log(a.bow); // a 타입은 Dog
  } else {
    console.log(a.meow); // a 타입은 Cat
  }
}
```

```typescript
const isRejected (input: PromiseSettledResult<unknown>): input is PromiseRejectedResult => {
  return input.status === 'rejected'
};
const isFulfilled = <T>(input: PromiseSettledResult<T>): input is PromiseFulfilledResult<T> => {
  return input.status === 'fulfilled'
};

const promises = await Promise.allSettled([Promise.resolve('a'), Promise.resolve('b')]);
// const errors = promises.filter((promise) => promise.status === 'rejected')
const errors = promises.filter(isRejected); // errors의 타입은 PromiseRejectedResult[]

```

---

## readonly

읽기 전용 속성 타입

```typescript
interface A {
  readonly name: string;
  age: number;
}

const customer: A = { name: "Jake", age: 21 };
customer.age += 1; // 통과
customer.name = "Mike"; // 타입 에러 : 읽기 전용 속성이므로 'name'에 할당할 수 없음
```

## index signature

정해진 구조에 대한 타입 정의

```typescript
type A = { [key: string]: number }; // 어떤 key가 string이면 값은 number다.
const aaa: A = { Human: 1, Mammal: 12, Animal: 10 };
```

## maped types

정해진 값들로 타입 정의

```typescript
type B = "Human" | "Mammal" | "Animal"; // interface는 객체 형식만 가능
type C = { [key in B]: number }; // B에 해당하는 key는 number다.
const ccc: C = { Human: 1, Mammal: 12, Animal: 10 };
```

---

## typescript의 Class

- implements :
- private :
- protected :

---

## never 타입

[TOAST UI: Never 타입](https://ui.toast.com/weekly-pick/ko_20220323)

- never 타입은 값의 공집합이다.
- any 타입을 포함하여 어떤 값도 가질 수 없다.

#### 그럼 never 타입이 왜 필요할까?

1. 값을 포함할 수 없는 빈 타입

   - 제네릭과 함수에서 허용되지 않는 매개변수
   - 호환되지 않는 타입들의 교차 타입
   - 빈 합집합

2. 실행이 끝날 때 호출자에게 제어를 반환하지 않는 함수의 반환 타입

   - ex. nodejs의 process.exit

3. 절대 도달할 수 없을 조건 타입

   - 무한 루프

4. 거부된 프로미스에서 처리된 값의 타입
   ```javascript
   const p = Promise.reject("foo"); // const p : Promise<never>
   ```

#### never의 특징

- never는 유니온 타입에서 사라짐
  ```javascript
  type res = never | string; // string
  ```
- never는 교차 타입에서 덮어씀
  ```javascript
  type res = never & string; // never
  ```

#### never 활용

1. 허용할 수 없는 함수 매개변수에 제한을 가한다.
2. switch, if-else 문의 모든 상황을 보장한다.

```javascript
declare let myNever : never
function fn(input: never){}
fn(myNever) // (O)

// 인자가 없거나 never타입이 아니면 타입 에러
fn(); // (X)
fn(1); // (X)
fn('foo'); // (X)

// default case로 활용
function unknownColor(x: never): never {
  throw new Error("unknown color");
}
type Color= 'red' | 'green'

function getColorName(c: string): string{
  switch(c){
    case 'red':
      return 'is red';
    case 'green':
      return 'is green';
    default:
      // string 인자는 never 타입이 아니기 때문에 case에 속하지 않는 모든 케이스에 적용가능
      return unKnownColor(c);
  }
}
```

3. 부분적 타이핑을 허용하지 않는다.

- A 또는 B 타입의 매개변수를 받을 때, 두 타입 모두 포함하고 싶지는 않다면?

```javascript
type A = {
  a: string
  b?: never
}
type B = {
  b: number
  a?: never
}
const input = {a: 'foo', b: 12}
function fn(arg: A|B):void
fn(input) // (X) A타입일 때는 b가 막히고, B타입일 때는 a가 막힘

```

4. 의도하지 않은 API 사용 방지

```javascript
type Read = {}
type Write = {}
declare const toWrite = Write

declare class MyCache<T, R>{
  put(val:T): boolean;
  get(): R;
}

const cache = new MyCache<Write, Read>();
cache.put(toWrite) // (o)

// never로 읽기 전용 캐시 만들기
declare class ReadOnlyCache<R> extends MyCache<never, R> {}

const readonlyCache = new ReadOnlyCache<Read>()
readonlyCache.put(data) // (X) never 타입에 걸려서 타입 에러

```

5. 유니온 타입 멤버 필터링

```javascript
type Foo = {
  name : 'foo'
  id: number
}
type Bar = {
  name : 'bar'
  id: number
}

type All = Foo | Bar
type ExtractTypeByName<T, G> = T extends {name: G} ? T : never
type ExtractType = ExtractTypeByName<All, 'foo'> // type:Foo
type ExtractType = ExtractTypeByName<All, 'bar'> // type:Bar
type ExtractType = ExtractTypeByName<All, 'cat'> // 타입 에러

```

6. 매핑된 타입의 키 필터링

7. 재어 흐름 분석의 좁은 타입

8. 호환되지 않는 타입의 불가능한 교차 타입 표시

---

# typescript v.4.8

## {}와 Object

- type이 {}이거나 Object라면, 모든 타입을 허용한다는 뜻이다.
- 단, 소문자인 object는 객체 타입을 의미함.
- typescript v4.8부터 unknown의 type이 {}로 체크됨

```typescript
const a: {} = "allow"; // 허용
const b: Object = 10; // 허용
const z: unknown = "hi"; // 허용

if (z) {
  z; // z의 타입은 v4.7에서는 unknown, v4.8에서는 {}로 나옴
}
```

---
