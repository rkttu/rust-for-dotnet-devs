# null 가능성과 선택성

C#에서는 값이 없거나 아직 초기화되지 않았음을 나타낼 때 `null`을
자주 사용합니다. 다음 예제를 살펴보겠습니다.

```csharp
int? some = 1;
int? none = null;
```

Rust에는 `null`이 없으므로 활성화할 nullable 문맥도 없습니다.
값이 없거나 선택 사항인 경우에는 [`Option<T>`][option]로 표현합니다.
위 C# 코드에 대응하는 Rust 코드는 다음과 같습니다.

```rust
let some: Option<i32> = Some(1);
let none: Option<i32> = None;
```

Rust의 `Option<T>`는 F#의 [`'T option`][opt.fs]과 거의 같습니다.

[opt.fs]: https://fsharp.github.io/fsharp-core-docs/reference/fsharp-core-option-1.html

## 선택적 값에 따른 제어 흐름

C#에서는 null 가능 값을 처리할 때 `if`/`else` 문으로 흐름을
제어할 수 있습니다.

```csharp
uint? max = 10;
if (max is { } someMax)
{
    Console.WriteLine($"The maximum is {someMax}."); // The maximum is 10.
}
```

Rust에서는 패턴 일치로 같은 동작을 구현할 수 있습니다.

```rust
let max = Some(10u32);
match max {
    Some(max) => println!("The maximum is {}.", max), // The maximum is 10.
    None => ()
}
```

`if let`을 사용하면 코드를 더 간결하게 작성할 수 있습니다.

```rust
let max = Some(10u32);
if let Some(max) = max {
    println!("The maximum is {}.", max); // The maximum is 10.
}
```

## null 조건부 연산자

C#의 null 조건부 연산자 `?.`와 `?[]`는 `null`을 다루기 편하게
해 줍니다. Rust에서는 `Option`의 중첩 여부에 따라 [`map`][optmap]
또는 [`and_then`][opt_and_then] 메서드를 사용할 수 있습니다. 다음
예제에서 대응 관계를 살펴보겠습니다.

```csharp
string? some = "Hello, World!";
string? none = null;
Console.WriteLine(some?.Length); // 13
Console.WriteLine(none?.Length); // (blank)

record Name(string FirstName, string LastName);
record Person(Name? Name);

{
    Person? person = new Person(new Name("John", "Doe"));
    Console.WriteLine(person1?.Name?.FirstName); // John
}

{
    Person? person = new Person(null);
    Console.WriteLine(person1?.Name?.FirstName); // (blank)
}

{
    Person? person = null;
    Console.WriteLine(person1?.Name?.FirstName); // (blank)
}
```

다음 예제에서는 다른 중첩 형태를 비교합니다.

```rust
let some: Option<String> = Some(String::from("Hello, World!"));
let none: Option<String> = None;
println!("{:?}", some.map(|s| s.len())); // Some(13)
println!("{:?}", none.map(|s| s.len())); // None

struct Name { first_name: String, last_name: String }
struct Person { name: Option<Name> }

let person: Option<Person> = Some(Person {
    name: Some(Name {
        first_name: "John".into(),
        last_name: "Doe".into(),
    }),
});
println!("{:?}", person.and_then(|p| p.name.map(|name| name.first_name))); // Some("John")

let person: Option<Person> = Some(Person { name: None });
println!("{:?}", person.and_then(|p| p.name.map(|name| name.first_name))); // None

let person: Option<Person> = None;
println!("{:?}", person.and_then(|p| p.name.map(|name| name.first_name))); // None
```

[앞 장의 오류 전파 절][err]에서 살펴본 `?` 연산자는 `Option`에도
사용할 수 있습니다. `None`을 만나면 함수에서 `None`을 반환하고,
`Some`이면 안의 값으로 계산을 계속합니다.

```rust
fn foo(optional: Option<i32>) -> Option<String> {
    let value = optional?;
    Some(value.to_string())
}
```

[err]: exception-handling.md#오류-전파

## null 병합 연산자

C#의 null 병합 연산자 `??`는 null 가능 값이 `null`일 때 기본값을
선택하는 데 사용합니다.

```csharp
int? some = 1;
int? none = null;
Console.WriteLine(some ?? 0); // 1
Console.WriteLine(none ?? 0); // 0
```

Rust에서는 [`unwrap_or`][unwrap-or]로 같은 동작을 구현할 수
있습니다.

```rust
let some: Option<i32> = Some(1);
let none: Option<i32> = None;
println!("{:?}", some.unwrap_or(0)); // 1
println!("{:?}", none.unwrap_or(0)); // 0
```

기본값을 계산하는 비용이 크다면 `unwrap_or_else`를 사용할 수
있습니다. 이 메서드는 클로저를 받아 기본값을 지연 계산합니다.

## null 허용 연산자

C#의 null 허용 연산자 `!`는 컴파일러의 정적 흐름 분석에만 영향을
주므로 Rust에 직접 대응하는 구문이 없습니다. Rust에서는 같은 역할의
구문이 필요하지 않습니다. 다만 [`unwrap`][opt_unwrap]은 값이 `None`이면
패닉을 일으킨다는 점에서 비슷한 결과를 낼 수 있습니다.
[`expect`][opt_expect]도 비슷하지만 사용자 정의 오류 메시지를
넣을 수 있습니다. 앞서 설명했듯 패닉은 복구할 수 없는 상황에
사용합니다.

[option]: https://doc.rust-lang.org/std/option/enum.Option.html
[optmap]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map
[opt_and_then]: https://doc.rust-lang.org/std/option/enum.Option.html#method.and_then
[unwrap-or]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or
[opt_unwrap]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap
[opt_expect]: https://doc.rust-lang.org/std/option/enum.Option.html#method.expect
