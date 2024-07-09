# Option\<T> and Result<T, E> enum types for JavaScript

If you need more Rust-like features, check out [derive-rust](https://www.npmjs.com/package/derive-rust)

## Option\<T>

```ts
const a: Option<string> = Some("hello");
const b: Option<string> = None(); 
const c = None<string>();

// match() method for getting value
a.match({
  None: () => "",
  Some: (hello) => hello + " world!"
})

c.match({
  None: () => "",
  Some: (v) => v
})

// ifLet() method for the short way

Some(10).ifLet((n) => Some(n), (n) => {
  console.log(n / 2)
})

// as Some is a function itself we can pass its ref

Some(10).ifLet(Some, (num) => {
  console.log(num / 2);
}, () => {
  console.log("None 10"); // you can call optional else expression
})

// Data validation

const opt = Option.from<number>(undefined);

const x2Num = opt.match({
  Some: (num) => num * 2,
  None: () => console.error("null or undefined")
});

```

## Result<T, E>

```ts
const fooOk: Result<{result: boolean}, {}> = Ok({result: true});
const fooErr: Result<{result: boolean}, {}> = Err({});

// Data validation

const result = Result.from<{data: Data[]}>(null);

const data = result.match({
  Err: () => console.error("null or undefined"),
  Ok: (value) => value.data
});

// Error interception
Result.fromNever(() => { throw "Thrown" }).unwrapErr() // Thrown

Result.fromPromise(Promise.reject("Rejected"))
  .then(r => r.unwrapErr()) // Rejected

Result.fromAsync(async () => await fetch("I love Rust"))
  .then(r => r.unwrapErr()) // Response error

```

## Implementing of Enum

```ts

interface MyEnumArms<T, A> {
  Foo(value: T): A,
  Bar(value: string): A,
  Baz(value: string): A,
}

class MyEnum<T = string> {
  static Foo = (value: string) => new MyEnum(self => self.variant.Foo, value);
  static Bar = () => new MyEnum(self => self.variant.Bar);
  static Baz = () => new MyEnum(self => self.variant.Baz);

  #self: {
    variant: string,
    value: T
  };

  variant = {
    Foo: (value: string) => value,
    Bar: () => {},
    Baz: () => {},
  }

  private constructor(impl: (self: MyEnum<T>) => Function, value?: T) {
    let variant = impl(this);

    this.self = {
      variant: variant.name, 
      value: variant(value) ?? variant.name as T
    }
  }

  match<A>(arms: MyEnumArms<T, A>): A {
    switch (this.self.variant) {
      case arms.Foo.name: return arms.Foo(this.self.value);
      case arms.Bar.name: return arms.Bar(arms.Bar.name);
      default: return arms.Baz(arms.Baz.name);
    }
  }
}

const fooMatch = MyEnum.Foo("foo").match({
  Foo: (foo) => `my ${foo} value`,
  Bar: (bar) => `my ${bar} value`,
  Baz: (baz) => `my ${baz} value`,
});

console.log({fooMatch}) // { fooMatch: 'my foo value' }

const bazMatch = MyEnum.Baz().match({
  Foo: (foo) => `my ${foo} value`,
  Bar: (bar) => `my ${bar} value`,
  Baz: (baz) => `my ${baz} value`,
});

console.log({bazMatch}) // { bazMatch: 'my Baz value' }

```

## Declarations

```ts

interface ResultArms<T, E, A> {
    Err(err: E): A;
    Ok(ok: T): A;
}

class Result<T, E> {
    #private;
    $ref: [T | E];
    private constructor();
    static Err<E, T>(value: E): Result<T, E>;
    static Ok<T, E>(value: T): Result<T, E>;
    static from<T>(value: T | null | undefined): Result<T, null | undefined>;
    static fromNever<T, E>(fn: () => T): Result<T, E>;
    static fromPromise<T, E>(promise: Promise<T>): Promise<Result<T, E>>;
    static fromAsync<T, E>(fn: () => Promise<T>): Promise<Result<T, E>>;
    match<A>(arms: ResultArms<T, E, A>): A;
    isErr(): boolean;
    isOk(): boolean;
    ok(): Option<T>;
    err(): Option<E>;
    unwrap(): T;
    unwrapOr(value: T): T;
    unwrapErr(): E;
    expect(message: string): T;
    intoOption(): Option<T>;
    map<F>(fn: (ok: T) => F): Result<F, E>;
    mapErr<F>(fn: (err: E) => F): Result<T, F>;
    flatten(): Result<T, E>;
    ifLet<F>(fn: (r: T | E) => Result<T, E>, ifExpr: (value: T | E) => F, elseExpr?: (value: T | E) => F): F;
}

interface OptionArms<T, A> {
    Some(value: T): A;
    None(): A;
}

class Option<T> {
    #private;
    private constructor();
    static None<T>(): Option<T>;
    static Some<T>(value: T): Option<T>;
    static from<T>(value: T | null | undefined): Option<T>;
    match<A>(arms: OptionArms<T, A>): A;
    unwrap(): T;
    unwrapOr(value: T): T;
    expect(message: string): T;
    isNone(): boolean;
    isSome(): boolean;
    intoResult<E>(error: E): Result<T, E>;
    map<F>(fn: (value: T) => F): Option<F>;
    flatten(): Option<T>;
    okOr<E>(err: E): Result<T, E>;
    okOrElse<E>(fn: () => E): Result<T, E>;
    ifLet<F>(fn: (opt: T) => Option<T>, ifExpr: (value: T) => F, elseExpr?: (value: T) => F): F;
}

```