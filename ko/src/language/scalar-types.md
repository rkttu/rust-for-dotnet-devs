# 스칼라 타입

다음 표는 Rust의 기본 타입과 C# 및 .NET의 대응 타입을 보여 줍니다.

| Rust    | C#        | .NET                      | 비고          |
| ------- | --------- | ------------------------- | ------------- |
| `bool`  | `bool`    | `Boolean`                 |               |
| `char`  | `char`    | `Char`                    | 주 1 참조     |
| `i8`    | `sbyte`   | `SByte`                   |               |
| `i16`   | `short`   | `Int16`                   |               |
| `i32`   | `int`     | `Int32`                   |               |
| `i64`   | `long`    | `Int64`                   |               |
| `i128`  |           | `Int128`                  |               |
| `isize` | `nint`    | `IntPtr`                  |               |
| `u8`    | `byte`    | `Byte`                    |               |
| `u16`   | `ushort`  | `UInt16`                  |               |
| `u32`   | `uint`    | `UInt32`                  |               |
| `u64`   | `ulong`   | `UInt64`                  |               |
| `u128`  |           | `UInt128`                 |               |
| `usize` | `nuint`   | `UIntPtr`                 |               |
| `f32`   | `float`   | `Single`                  |               |
| `f64`   | `double`  | `Double`                  |               |
|         | `decimal` | `Decimal`                 |               |
| `()`    | `void`    | `Void` 또는 `ValueTuple`  | 주 2, 3 참조  |
|         | `object`  | `Object`                  | 주 3 참조     |

주석:

1. Rust의 [`char`][char.rs]와 .NET의 [`Char`][char.net]는 정의가
   다릅니다. Rust의 `char`는 4바이트 너비의 [유니코드 스칼라 값]입니다.
   .NET의 `Char`는 2바이트 너비이며 문자를 UTF-16으로 저장합니다.
   자세한 내용은 [Rust `char` 문서][char.rs]에서 확인할 수 있습니다.

2. Rust의 단위 타입 `()`는 값으로 표현할 수 있는 빈 튜플입니다. C#에서
   가장 가까운 개념은 값이 없음을 나타내는 `void`입니다. 다만 포인터와
   안전하지 않은 코드를 사용하는 경우를 제외하면 `void`를 값으로 표현할
   수 없습니다. .NET의 [`ValueTuple`][ValueTuple]도 빈 튜플이지만
   C#에는 이를 나타내는 `()` 같은 리터럴 구문이 없습니다. C#에서
   `ValueTuple`을 사용할 수는 있으나 흔하지 않습니다. [F#에는
   Rust와 비슷한 단위 타입이 있습니다][unit.fs].

3. `void`와 `object`는 스칼라 타입이 아닙니다. .NET 타입 계층에서
   `int`와 같은 스칼라 타입이 `object`의 하위 타입이라는 점과는
   별개입니다. 비교의 편의를 위해 표에 함께 넣었습니다.

관련 자료:

- [기본 타입(Rust By Example)][primitives.rs]

[char.net]: https://learn.microsoft.com/en-us/dotnet/api/system.char
[char.rs]: https://doc.rust-lang.org/std/primitive.char.html
[유니코드 스칼라 값]: https://www.unicode.org/glossary/#unicode_scalar_value
[ValueTuple]: https://learn.microsoft.com/en-us/dotnet/api/system.valuetuple?view=net-7.0
[unit.fs]: https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/unit-type
[primitives.rs]: https://doc.rust-lang.org/rust-by-example/primitives.html
