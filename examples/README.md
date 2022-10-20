# type-challenges

## pick

type T로부터 프로퍼티 K를 pick하여 타입을 구성하라.

```typescript
// your code
type MyPick<T, K> = any;

interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = MyPick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
```

```typescript
//answer
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

- K는 T의 key를 상속받는다.
- K의 프로퍼티인 P를 key로 갖는 타입은 T의 P 프로퍼티 타입이다.

## readonly

T의 key type은 readonly이다.

```typescript
//your code
type MyReadonly<T> = any;

// examples
interface Todo {
  title: string;
  description: string;
}

const todo: MyReadonly<Todo> = {
  title: "Hey",
  description: "foobar",
};

todo.title = "Hello"; // Error: cannot reassign a readonly property
todo.description = "barFoo"; // Error: cannot reassign a readonly property
```

```typescript
//answer
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

- T의 key에 해당하는 type은 readonly이다.

## Tuple to Object

배열 타입을 Object 타입으로 변경하라.

```typescript
// your code
type TupleToObject<T extends readonly any[]> = any;

const tuple = ["tesla", "model 3", "model X", "model Y"] as const;

type result = TupleToObject<typeof tuple>; // expected { tesla: 'tesla', 'model 3': 'model 3', 'model X': 'model X', 'model Y': 'model Y'}

// @ts-expect-error
type error = TupleToObject<[[1, 2], {}]>;
```

```typescript
// answer
type TupleToObject<T extends readonly (string | number | symbol)[]> = {
  [P in T[number]]: P;
};
```

- T는 typeof tuple이므로, ['tesla', ...] 와 같은 상수 튜플 타입
- T[number]는 'tesla' | 'model 3' | 'model X' | 'model Y'
- P in T[number]에서 P는 'tesla'와 같은 key 값
- 특정 P에 P 타입을 대입
- 최종 타입 : {'tesla' : 'tesla', 'model 3' : 'model 3', ...}
