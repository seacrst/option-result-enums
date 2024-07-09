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

## Create your own Enum

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

match(MyEnum.Foo("hello"), () => [
  (hello) => [MyEnum.Foo(hello), () => console.log(hello + " world")],
  (bar) => [MyEnum.Bar(), () => console.log("hello " + bar)]
], (_, hello) => console.log(hello))

// hello world
// ________________

match(MyEnum.Bar(), () => [
  (hello) => [MyEnum.Foo(hello), () => console.log(hello + " world")],
  (bar) => [MyEnum.Bar(), () => console.log("hello " + bar.toLowerCase())]
], (_, hello) => console.log(hello))

// hello bar
// ________________

match(MyEnum.Baz(), () => [
  (hello) => [MyEnum.Foo(hello), () => console.log(hello + " world")],
  (bar) => [MyEnum.Bar(), () => console.log("hello " + bar.toLowerCase())]
], (_, Baz) => console.log(Baz.toLowerCase()))

// baz

```