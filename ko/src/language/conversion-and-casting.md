# 변환과 캐스팅

C#과 Rust는 모두 컴파일할 때 타입을 정적으로 검사합니다. 변수를 선언한
뒤에는 해당 타입으로 암시적으로 변환할 수 없는 다른 타입의 값을 할당할
수 없습니다. C#의 여러 타입 변환 방식에는 Rust에서 대응하는 방법이
있습니다.

## 암시적 변환

C#과 Rust 모두 암시적 변환을 지원합니다. Rust에서는 이를 [타입 강제
변환]이라고 부릅니다. 다음 예제를 살펴보겠습니다.

```csharp
int intNumber = 1;
long longNumber = intNumber;
```

Rust는 허용하는 타입 강제 변환의 범위를 훨씬 좁게 제한합니다.

```rust
let int_number: i32 = 1;
let long_number: i64 = int_number; // error: expected `i64`, found `i32`
```

[하위 타입 관계][subtyping.rs]를 이용한 올바른 암시적 변환 예제는
다음과 같습니다.

```rust
fn bar<'a>() {
    let s: &'static str = "hi";
    let t: &'a str = s;
}
```

관련 자료:

- [역참조 강제 변환]
- [하위 타입과 변성]

[타입 강제 변환]: https://doc.rust-lang.org/reference/type-coercions.html
[subtyping.rs]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md#subtyping
[역참조 강제 변환]: https://doc.rust-lang.org/std/ops/trait.Deref.html#more-on-deref-coercion
[하위 타입과 변성]: https://doc.rust-lang.org/reference/subtyping.html#subtyping-and-variance

## 명시적 변환

정보를 잃을 수 있는 변환에는 C#에서 캐스팅 식을 사용한 명시적
변환이 필요합니다.

```csharp
double a = 1.2;
int b = (int)a;
```

명시적 변환은 다운캐스팅 과정에서 `OverflowException`이나
`InvalidCastException` 같은 예외를 발생시키며 실행 중 실패할 수
있습니다.

Rust는 기본 타입 간 강제 변환 대신 [`as`][as.rs] 키워드를 사용한
[명시적 변환][casting.rs], 즉 캐스팅을 제공합니다. Rust의 캐스팅은
패닉을 일으키지 않습니다.

```rust
let int_number: i32 = 1;
let long_number: i64 = int_number as _;
```

[casting.rs]: https://doc.rust-lang.org/rust-by-example/types/cast.html
[as.rs]: https://doc.rust-lang.org/reference/expressions/operator-expr.html#type-cast-expressions

## 사용자 정의 변환

.NET 타입에는 한 타입을 다른 타입으로 바꾸는 사용자 정의 변환
연산자를 둘 수 있습니다. `System.IConvertible`도 타입 간 변환에
사용합니다.

Rust 표준 라이브러리는 [`From`][from.rs] 트레이트와 그 역방향인
[`Into`][into.rs] 트레이트로 값 변환을 추상화합니다. 어떤 타입에
`From`을 구현하면 `Into`의 기본 구현도 자동으로 제공됩니다. Rust에서는
이를 _일괄 구현_이라고 부릅니다. 다음 예제는 두 가지 타입 변환을
보여 줍니다.

```rust
fn main() {
    let my_id = MyId("id".into()); // `into()` is implemented automatically due to the `From<&str>` trait implementation for `String`.
    println!("{}", String::from(my_id)); // This uses the `From<MyId>` implementation for `String`.
}

struct MyId(String);

impl From<MyId> for String {
    fn from(MyId(value): MyId) -> Self {
        value
    }
}
```

관련 자료:

- 변환에 실패할 수 있는 [`TryFrom`][try-from.rs]과 [`TryInto`][try-into.rs]

[from.rs]: https://doc.rust-lang.org/std/convert/trait.From.html
[into.rs]: https://doc.rust-lang.org/std/convert/trait.Into.html
[try-from.rs]: https://doc.rust-lang.org/std/convert/trait.TryFrom.html
[try-into.rs]: https://doc.rust-lang.org/std/convert/trait.TryInto.html
