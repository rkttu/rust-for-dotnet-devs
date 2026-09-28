# 변수

다음 C# 예제는 변수에 값을 할당합니다.

```csharp
int x = 5;
```

Rust에서는 다음과 같이 작성합니다.

```rust
let x: i32 = 5;
```

여기까지 두 언어의 눈에 띄는 차이는 타입 선언의 위치뿐입니다. C#과 Rust는
모두 타입 안전성을 보장합니다. 컴파일러는 변수에 지정된 타입의 값만
저장하도록 검사합니다. 컴파일러의 타입 추론을 활용하면 예제를 더 짧게
작성할 수 있습니다. 먼저 C# 코드입니다.

```csharp
var x = 5;
```

Rust 코드는 다음과 같습니다.

```rust
let x = 5;
```

첫 예제에서 변수에 값을 다시 할당하면 두 언어의 동작이 달라집니다.

```csharp
var x = 5;
x = 6;
Console.WriteLine(x); // 6
```

Rust에서 같은 문장을 작성하면 컴파일되지 않습니다.

```rust
let x = 5;
x = 6; // Error: cannot assign twice to immutable variable 'x'.
println!("{}", x);
```

Rust의 변수는 기본적으로 _불변_입니다. 이름에 값을 바인딩한 뒤에는
그 값을 변경할 수 없습니다. 변수 이름 앞에 [`mut`][mut.rs]를 붙이면
_가변_ 변수를 만들 수 있습니다.

```rust
let mut x = 5;
x = 6;
println!("{}", x); // 6
```

또는 변수 _섀도잉_을 사용하면 가변성 없이도 예제를 수정할 수 있습니다.

```rust
let x = 5;
let x = 6;
println!("{}", x); // 6
```

C#도 섀도잉을 지원합니다. 예를 들어 로컬 변수가 필드를 가리거나
파생 타입의 멤버가 기본 타입의 멤버를 가릴 수 있습니다. Rust에서는
위 예제처럼 같은 이름을 유지하면서 변수 타입도 바꿀 수 있습니다.
데이터를 여러 타입과 형태로 변환할 때마다 새 이름을 정하지 않아도
된다는 장점이 있습니다.

관련 자료:

- 가변성의 영향에 관한 [데이터 레이스와 경쟁 상태]
- [범위와 섀도잉]
- _이동_과 _소유권_에 관한 [메모리 관리][memory-management-section]

[mut.rs]: https://doc.rust-lang.org/std/keyword.mut.html
[memory-management-section]: ../memory-management/index.md
[데이터 레이스와 경쟁 상태]: https://doc.rust-lang.org/nomicon/races.html
[범위와 섀도잉]: https://doc.rust-lang.org/stable/rust-by-example/variable_bindings/scope.html#scope-and-shadowing
