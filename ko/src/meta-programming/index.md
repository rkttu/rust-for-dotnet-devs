# 메타프로그래밍

메타프로그래밍은 다른 코드를 작성하거나 생성하는 코드를 만드는 기법입니다.

C#에서는 .NET 5부터 Roslyn의 [소스 생성기][source-gen]를 메타프로그래밍에 사용할 수 있습니다. 소스 생성기는 빌드 중 C# 소스 파일을 만들고 이를 사용자 코드의 컴파일에 추가합니다. 소스 생성기가 도입되기 전에도 Visual Studio는 [T4 텍스트 템플릿][T4]을 통한 코드 생성을 지원했습니다. [템플릿 예제][template]와 [생성 결과][concretization]에서 작동 방식을 살펴볼 수 있습니다.

Rust는 [매크로][macros]로 메타프로그래밍을 지원합니다. 매크로는 선언적 매크로와 절차적 매크로로 나뉩니다.

선언적 매크로는 입력 코드를 패턴과 비교하고 일치하는 패턴에 대응하는 코드를 생성합니다. 다음은 `println!("Some text")`처럼 호출할 수 있는 `println!` 매크로의 정의입니다.

```rust
macro_rules! println {
    () => {
        $crate::print!("\n")
    };
    ($($arg:tt)*) => {{
        $crate::io::_print($crate::format_args_nl!($($arg)*));
    }};
}
```

선언적 매크로 작성 방법은 Rust 레퍼런스의 [예제 기반 매크로][macros by example]와 [The Little Book of Rust Macros]에서 자세히 다룹니다.

[절차적 매크로][procedural macros]는 코드를 입력받아 처리한 뒤 새로운 코드를 출력합니다.

C#에서 메타프로그래밍에 사용하는 또 다른 기법으로 리플렉션이 있습니다. Rust는 리플렉션을 지원하지 않습니다.

[source-gen]: https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview

## 함수형 매크로

함수형 매크로는 `function!(...)` 형태로 호출합니다. 다음 예제의 `print_something` 매크로는 `"Something"`을 출력하는 `print_it` 함수를 생성합니다.

`lib.rs`에 다음 코드를 작성합니다.

```rust
extern crate proc_macro;
use proc_macro::TokenStream;

#[proc_macro]
pub fn print_something(_item: TokenStream) -> TokenStream {
    "fn print_it() { println!(\"Something\") }".parse().unwrap()
}
```

`main.rs`에서는 다음처럼 매크로를 사용합니다.

```rust
use replace_crate_name_here::print_something;
print_something!();

fn main() {
    print_it();
}
```

## 파생 매크로

파생 매크로는 구조체, 열거형, 공용체의 토큰 스트림을 받아 새 항목을 생성할 수 있습니다. `#[derive(Clone)]`은 입력 타입이 `Clone` 트레이트를 구현하는 데 필요한 코드를 생성하는 파생 매크로의 예입니다.

사용자 정의 파생 매크로의 작성 방법은 Rust 레퍼런스의 [파생 매크로][derive macros] 절을 참고할 수 있습니다.

[derive macros]: https://doc.rust-lang.org/reference/procedural-macros.html#derive-macros

## 특성 매크로

특성 매크로는 Rust 항목에 붙일 수 있는 새로운 특성을 정의합니다. Tokio로 비동기 코드를 작성할 때는 다음처럼 `#[tokio::main]` 특성 매크로를 비동기 진입점에 붙일 수 있습니다.

```rust
#[tokio::main]
async fn main() {
    println!("Hello world");
}
```

사용자 정의 특성 매크로의 작성 방법은 Rust 레퍼런스의 [특성 매크로][attribute macros] 절을 참고할 수 있습니다.

[attribute macros]: https://doc.rust-lang.org/reference/procedural-macros.html#attribute-macros

[T4]: https://learn.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2015/modeling/code-generation-and-t4-text-templates?view=vs-2015&redirectedfrom=MSDN
[template]: https://github.com/atifaziz/Jacob/blob/master/src/JsonReader.g.tt
[concretization]: https://github.com/atifaziz/Jacob/blob/master/src/JsonReader.g.cs
[macros]: https://doc.rust-lang.org/book/ch19-06-macros.html
[macros by example]: https://doc.rust-lang.org/reference/macros-by-example.html
[procedural macros]: https://doc.rust-lang.org/reference/procedural-macros.html
[The Little Book of Rust Macros]: https://veykril.github.io/tlborm/
