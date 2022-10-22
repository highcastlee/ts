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
