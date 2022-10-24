# utility-types

## Partial, Pick, Omit

#### Partial<T>

부분적 타입 사용
전체를 `?:` 처리

```typescript
interface Profile {
  name: string;
  age: number;
}

const person: Profile = {
  name: "ai",
  age: 10,
};

const person2: Partial<Profile> = {
  name: "gi",
  // age?: number 처리됨
};
```

#### Pick<T, ...types>

특정 타입 선별적 사용
사용하고자 하는 타입만 따로 설정

```typescript
type Pick<T, S> =

const person: Pick<Profile, "name"> = {
  name: "jay",
};
```

#### Omit<T, ...types>

특정 타입 제외 나머지 사용

```typescript
type Omit<T, S extends keyof any> = Pick<T, Exclude<keyof T, S>>;

const person: Omit<Profile, "age"> = {
  name: "jay",
};
```

---

## Required, Record, NonNullable

#### Required<T>

옵셔널한 타입들을 필수로 변경시킴
`-`키워드는 옵셔널 `?`를 제거하는 역할

```typescript
type Required<T> = {
  [key in keyof T]-?: T[key];
};

type Profile = {
  name?: string;
  age?: number;
};

const person: Required<Profile> = {
  name: "jay",
  age: 10,
};
```

#### Record<T,S>

key가 T이고, 값이 S인 객체 타입

```typescript
type Record<T extends keyof any, S> = {
  [key in T]: T[key];
};
const names = "jay" | "kai";
const person: Record<names, number> = {
  jay: 10,
  kai: 123,
};
```

#### NonNullable

null이 될 수 없는 값만 사용

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type B = null | undefined | boolean | string;
type A = NonNullable<B>; // boolean | string
```

---

## infer

infer는 타입을 추론하여 네이밍할 때 사용한다.
infer는 조건문에 쓰이는 타입 중 하나를 이름 붙여서 빼 와서, 삼항 연산자의 true절이나 false절에 사용하기 위해 사용한다.
**infer는 extends, 조건문에서만 사용할 수 있다.**

#### ReturnType<T>

함수 타입을 인자 T로 받아서, 어떠한 조건문을 거쳐 반환 타입 R이나 any를 내놓는다.

```typescript
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

`T extends (...args : any) => any` 조건은 T가 함수 타입이어야한다는 조건이다.

`T extends (...args:any) => infer R`이 참이면, `R` 타입을 반환, 아니면 `any`를 반환한다.

즉, `T는 함수여야하는데, 그 반환 타입을 R이라고 할 것이다.`

#### Parameters<T>

함수 타입을 인자 T로 받아서, 해당 함수의 파라미터 타입을 반환한다.

```typescript
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : any;
```

파라미터의 타입을 P로 네이밍하여 조건이 truthy이면 P 반환

#### ConstructorParameters<T>

class의 constructor parameter 타입

```typescript
// 생성자의 파라미터 타입
type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : any;
```

**예시**
클래스는 그냥 타입으로 쓸 수도 있는데,
typeof A는 클래스의 생성자, 그냥 class A 타입은 인스턴스 타입이다.

```typescript
class A {
  a: string;
  b: number;
  constructor(a: string, b: number) {
    this.a = a;
    this.b = b;
  }
}
const c = new A("abc", 123);
type C = ConstructorParameters<typeof A>; // {a: string, b: number}
type I = InstanceType<typeof A>;
```
