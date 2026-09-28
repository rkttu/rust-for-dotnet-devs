# 문서 주석

C#은 XML 텍스트를 담는 주석 구문으로 타입의 API를 문서화합니다.
C# 컴파일러는 주석과 API 시그니처를 구조화한 XML 파일을 만듭니다.
다른 도구는 이 파일을 처리해 사람이 읽기 좋은 문서를 생성할 수
있습니다. 다음은 간단한 C# 예제입니다.

```csharp
/// <summary>
/// <c>MyClass</c>에 대한 문서 주석입니다.
/// </summary>
public class MyClass {}
```

Rust의 [문서 주석]은 C# 문서 주석에 대응합니다. Rust 문서 주석에는
Markdown 구문을 사용합니다. Rust 문서 컴파일러인 [`rustdoc`][rustdoc]은
보통 [`cargo doc`][cargo doc]을 통해 실행하며 주석을 문서로
컴파일합니다. 다음 예제를 살펴보겠습니다.

```rust
/// `MyStruct`에 대한 문서 주석입니다.
struct MyStruct;
```

.NET SDK에는 `cargo doc`에 대응하는 `dotnet doc` 같은 명령이
없습니다.

관련 자료:

- [문서 작성 방법]
- [문서 테스트]

[문서 주석]: https://doc.rust-lang.org/rust-by-example/meta/doc.html
[rustdoc]: https://doc.rust-lang.org/rustdoc/index.html
[cargo doc]: https://doc.rust-lang.org/cargo/commands/cargo-doc.html
[문서 작성 방법]: https://doc.rust-lang.org/rustdoc/how-to-write-documentation.html
[문서 테스트]: https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html
