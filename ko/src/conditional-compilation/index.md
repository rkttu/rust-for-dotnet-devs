# 조건부 컴파일

.NET과 Rust는 모두 외부 조건에 따라 특정 코드를 컴파일할 수 있습니다.

.NET에서는 [전처리기 지시문][preproc-dir]으로 조건부 컴파일을 제어합니다.

```csharp
#if debug
    Console.WriteLine("Debug");
#else
    Console.WriteLine("Not debug");
#endif
```

미리 정의된 기호 외에도 컴파일러 옵션인 _[DefineConstants]_로 기호를 정의할 수 있습니다. 이 기호를 `#if`, `#else`, `#elif`, `#endif`와 함께 사용하여 소스 파일을 조건에 따라 컴파일합니다.

Rust에서는 [`cfg` 특성][cfg], [`cfg_attr` 특성][cfg-attr], [`cfg!` 매크로][cfg-macro]로 조건부 컴파일을 제어합니다. .NET과 마찬가지로 [컴파일러 플래그 `--cfg`][cfg-flag]를 사용하여 구성 옵션을 직접 설정할 수도 있습니다.

[`cfg` 특성][cfg]은 구성 조건식(`ConfigurationPredicate`)을 평가하여 코드의 포함 여부를 결정합니다.

```rust
use std::fmt::{Display, Formatter};

struct MyStruct;

// This implementation of Display is only included when the OS is unix but foo is not equal to bar
// You can compile an executable for this version, on linux, with 'rustc main.rs --cfg foo=\"baz\"'
#[cfg(all(unix, not(foo = "bar")))]
impl Display for MyStruct {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        f.write_str("Running without foo=bar configuration")
    }
}

// This function is only included when both unix and foo=bar are defined
// You can compile an executable for this version, on linux, with 'rustc main.rs --cfg foo=\"bar\"'
#[cfg(all(unix, foo = "bar"))]
impl Display for MyStruct {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        f.write_str("Running with foo=bar configuration")
    }
}

// This function is panicking when not compiled for unix
// You can compile an executable for this version, on windows, with 'rustc main.rs'
#[cfg(not(unix))]
impl Display for MyStruct {
    fn fmt(&self, _f: &mut Formatter<'_>) -> std::fmt::Result {
        panic!()
    }
}

fn main() {
    println!("{}", MyStruct);
}
```

[`cfg_attr` 특성][cfg-attr]은 구성 조건식에 따라 다른 특성을 적용합니다.

```rust
#[cfg_attr(feature = "serialization_support", derive(Serialize, Deserialize))]
pub struct MaybeSerializableStruct;

// When the `serialization_support` feature flag is enabled, the above will expand to:
// #[derive(Serialize, Deserialize)]
// pub struct MaybeSerializableStruct;
```

내장 [`cfg!` 매크로][cfg-macro]는 하나의 구성 조건식을 받아 참이면 `true`, 거짓이면 `false`로 평가합니다.

```rust
if cfg!(unix) {
  println!("I'm running on a unix machine!");
}
```

관련 내용은 [조건부 컴파일][conditional-compilation] 문서에서 확인할 수 있습니다.

## 기능 플래그

조건부 컴파일은 선택적 의존성을 제공할 때도 유용합니다. Cargo에서는 패키지의 `Cargo.toml` 파일에 있는 `[features]` 테이블에 기능 이름을 정의합니다. 각 기능은 활성화하거나 비활성화할 수 있습니다. 빌드할 패키지의 기능은 `--features` 같은 명령줄 옵션으로 활성화합니다. 의존 패키지의 기능은 `Cargo.toml`의 의존성 선언에서 활성화합니다.

자세한 내용은 Cargo의 [기능 플래그 문서][features]를 참고할 수 있습니다.

[features]: https://doc.rust-lang.org/cargo/reference/features.html
[conditional-compilation]: https://doc.rust-lang.org/reference/conditional-compilation.html#conditional-compilation
[cfg]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-attribute
[cfg-flag]: https://doc.rust-lang.org/rustc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[cfg-attr]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg_attr-attribute
[cfg-macro]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-macro
[preproc-dir]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/preprocessor-directives#conditional-compilation
[DefineConstants]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/language#defineconstants
