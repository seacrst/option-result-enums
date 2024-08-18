# Option\<T> and Result<T, E> enum types for JavaScript

If you need more Rust-like features, check out [derive-rust](https://www.npmjs.com/package/derive-rust)

## Option\<T>

```ts
const a = Some("hello");
const b: Option<string> = None(); 
const c = None<string>();

const value = a.match({
  None: () => "",
  Some: (hello) => hello + " world!"
});

console.log(value) // hello world!

c.match({
  None: () => "",
  Some: (v) => v
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

## Declarations

```ts
class Option<T> {
    #private;
    private constructor();
    static None<T>(): Option<T>;
    static Some<T>(value: T): Option<T>;
    static from<T>(value: T | null | undefined): Option<T>;
    match<A>(option: OptionArms<T, A>): A;
    unwrap(): T;
    unwrapOr(value: T): T;
    unwrapOrElse(f: () => T): T;
    expect(message: string): T;
    inspect(f: (value: T) => any): Option<T>;
    insert(value: T): T;
    getOrInsert(value: T): T;
    getOrInsertWith(f: () => T): T;
    isNone(): boolean;
    isSome(): boolean;
    isSomeAnd(f: (value: T) => boolean): boolean;
    intoResult<E>(error: E): Result<T, E>;
    map<F>(predicate: (value: T) => F): Option<F>;
    mapOr<V>(value: V, f: (value: T) => V): V;
    mapOrElse<V>(defaultF: () => V, f: (value: T) => V): V;
    filter(predicate: (value: T) => boolean): Option<T>;
    flatten(): Option<T>;
    take(): Option<T>;
    takeIf(predicate: (value: T) => boolean): Option<T>;
    replace(value: T): Option<T>;
    zip<U>(other: Option<U>): Option<[T, U]>;
    zipWith<O, R>(other: Option<O>, f: (self: T, other: O) => R): Option<R>;
    unzip<U>(): [Option<T>, Option<U>];
    transpose<E>(): Result<Option<T>, E>;
    okOr<E>(err: E): Result<T, E>;
    okOrElse<E>(err: () => E): Result<T, E>;
    and<U>(option: Option<U>): Option<U>;
    andThen<U>(f: (value: T) => Option<U>): Option<U>;
    or(option: Option<T>): Option<T>;
    orElse(f: () => Option<T>): Option<T>;
    xor(option: Option<T>): Option<T>;
    [Symbol.iterator](): Generator<T | undefined, void, unknown>;
    iter(): (T | undefined)[];
}

class Result<T, E> {
    #private;
    private constructor();
    static Err<E, T>(value: E): Result<T, E>;
    static Ok<T, E>(value: T): Result<T, E>;
    static from<T>(value: T | null | undefined): Result<T, null | undefined>;
    static fromNever<T, E>(fn: () => T): Result<T, E>;
    static fromPromise<T, E>(promise: Promise<T>): Promise<Result<T, E>>;
    static fromAsync<T, E>(fn: () => Promise<T>): Promise<Result<T, E>>;
    match<A>(arms: ResultArms<T, E, A>): A;
    isErr(): boolean;
    isErrAnd(f: (value: E) => boolean): boolean;
    isOk(): boolean;
    isOkAnd(f: (value: T) => boolean): boolean;
    ok(): Option<T>;
    err(): Option<E>;
    unwrap(): T;
    unwrapOr(value: T): T;
    unwrapOrElse(f: (err: E) => T): T;
    unwrapErr(): E;
    expect(message: string): T;
    intoOk(): T;
    intoErr(): E;
    intoOption(): Option<T>;
    map<F>(predicate: (ok: T) => F): Result<F, E>;
    mapOr<V>(value: V, f: (ok: T) => V): V;
    mapOrElse<F, V>(defaultF: (err: E) => V, f: (ok: T) => V): V;
    mapErr<F>(predicate: (err: E) => F): Result<T, F>;
    flatten(): Result<T, E>;
    transpose(): Option<Result<T, E>>;
    inspect(f: (ok: T) => any): Result<T, E>;
    inspectErr(f: (err: E) => any): Result<T, E>;
    and<U>(res: Result<U, E>): Result<U, E>;
    andThen<U>(f: (ok: T) => Result<U, E>): Result<U, E>;
    or<F>(res: Result<T, F>): Result<T, F>;
    orElse<F>(f: (err: E) => Result<T, F>): Result<T, F>;
    [Symbol.iterator](): Generator<T | E, void, unknown>;
    iter(): (T | E)[];
}

```