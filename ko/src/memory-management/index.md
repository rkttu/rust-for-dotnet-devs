# 메모리 관리

Rust는 C#과 .NET처럼 _메모리 안전성_을 제공해 메모리 접근 오류를
예방합니다. 이런 오류는 소프트웨어 보안 취약점의 흔한 원인입니다.
Rust는 CLR 같은 런타임의 검사 없이 컴파일할 때 메모리 안전성을
보장합니다. 예외적으로 배열 범위 검사는 컴파일된 코드가 실행 중에
수행합니다. .NET의 JIT 컴파일 코드도 같은 검사를 수행합니다.
C#처럼 Rust에서도 [안전하지 않은 코드를 작성할 수 있습니다][unsafe-rust].
두 언어는 메모리 안전성을 보장하지 않는 함수와 코드 블록에 모두
`unsafe` 키워드를 사용합니다.

[unsafe-rust]: https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html

Rust에는 가비지 수집기(GC)가 없습니다. 메모리 관리는 개발자의
책임입니다. 다만 _안전한 Rust_의 소유권 규칙은 메모리를 더 이상
사용하지 않을 때 해제하도록 합니다. 예를 들어 블록이나 함수를
벗어나면 해당 메모리가 해제될 수 있습니다. 컴파일러는 정적 분석으로
[소유권] 규칙을 검사하고 위반한 코드를 컴파일 오류로 거부합니다.

[소유권]: https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html

.NET에는 정적 필드, 스레드 스택의 로컬 변수, CPU 레지스터, 핸들
등의 GC 루트를 제외하면 Rust와 같은 메모리 소유권 개념이 없습니다.
GC는 수집할 때 루트에서 참조를 따라 사용 중인 메모리를 찾아내고
나머지를 정리합니다. 대부분의 .NET 코드는 소유권이나 GC의 동작을
의식하지 않고 작성할 수 있습니다. 다만 성능에 민감한 코드에서는
힙에 할당하는 객체의 양과 속도를 고려할 수 있습니다.

반면 Rust의 소유권 규칙은 함수, 타입, 자료구조의 설계부터 코드
작성까지 영향을 줍니다. 개발자는 소유권을 명시적으로 고려해야
합니다. Rust는 데이터 사용에 엄격한 규칙을 적용해 실행 중 발생할
수 있는 데이터 [경쟁 상태]와 손상 문제도 컴파일할 때 찾아냅니다.
이 장에서는 스레드 안전성보다 메모리 관리와 소유권에 집중합니다.

[경쟁 상태]: https://doc.rust-lang.org/nomicon/races.html

Rust에서는 어느 시점에든 스택 또는 힙에 있는 구조체의 메모리를
소유하는 주체가 하나뿐입니다. 컴파일러는 [수명][lifetimes.rs]과
소유권을 추적합니다. 소유권을 다른 곳으로 넘기는 동작을 _이동_이라고
부릅니다. 다음 Rust 예제에서 이를 확인할 수 있습니다.

[lifetimes.rs]: https://doc.rust-lang.org/rust-by-example/scope/lifetime.html

```rust
#![allow(dead_code, unused_variables)]

struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let a = Point { x: 12, y: 34 }; // point owned by a
    let b = a;                      // b owns the point now
    println!("{}, {}", a.x, a.y);   // compiler error!
}
```

`main`의 첫 문장은 `Point`를 만들고 `a`에 소유권을 부여합니다.
두 번째 문장에서 소유권이 `a`에서 `b`로 이동하므로 `a`는 더
이상 유효한 메모리를 나타내지 않습니다. 마지막 문장에서 `a`를
통해 좌표를 출력하려 하면 컴파일에 실패합니다. 다음처럼 `main`을
수정할 수 있습니다.

```rust
fn main() {
    let a = Point { x: 12, y: 34 }; // point owned by a
    let b = a;                      // b owns the point now
    println!("{}, {}", b.x, b.y);   // ok, uses b
}   // point behind b is dropped
```

`main`이 끝나면 `a`와 `b`가 범위를 벗어납니다. 스택이 `main`
호출 전 상태로 돌아가면서 `b`가 가리키던 메모리를 해제합니다.
Rust에서는 `b`가 소유한 점이 _드롭_되었다고 말합니다. `a`는 이미
소유권을 넘겼으므로 범위를 벗어나도 드롭할 점이 없습니다.

Rust 구조체는 [`Drop`][drop.rs] 트레이트를 구현해 인스턴스가
드롭될 때 실행할 코드를 정의할 수 있습니다.

[drop.rs]: https://doc.rust-lang.org/std/ops/trait.Drop.html

C#에서 드롭과 대략 비슷한 기능으로 클래스 [종료자]가 있습니다.
GC는 나중에 종료자를 자동 호출하지만 Rust의 드롭은 컴파일러가
범위와 수명을 근거로 소유자가 없어졌다고 판단한 위치에서 즉시,
결정적으로 실행합니다. .NET에서 `Drop`과 가까운 인터페이스는
[`IDisposable`][IDisposable]입니다. 타입은 이를 구현해 보유한
비관리 리소스나 메모리를 해제합니다. .NET이 결정적 해제를 강제하는
것은 아니지만 C#의 `using` 문을 사용하면 블록이 끝날 때 인스턴스를
해제할 수 있습니다.

[종료자]: https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers
[IDisposable]: https://learn.microsoft.com/en-us/dotnet/api/system.idisposable

Rust의 `'static`은 전역 수명을 나타내는 예약된 수명 지정자입니다.
C#의 타입에 선언한 정적 _읽기 전용_ 필드가 아주 거친 비유에
해당합니다.

C#과 .NET에서는 참조를 자유롭게 공유하므로 단일 소유자와 소유권
이동이 제한적으로 보일 수 있습니다. Rust에서도 [`Rc`][rc.rs]
스마트 포인터로 _공유 소유권_을 구현할 수 있습니다. `Rc`는 참조
횟수를 셉니다. [스마트 포인터를 복제할 때][Rc::clone] 횟수가
증가하고 복제본이 드롭되면 감소합니다. 참조 횟수가 0이 되면
스마트 포인터 뒤의 인스턴스가 드롭됩니다. 다음 예제는 앞 예제를
확장해 이 동작을 보여 줍니다.

[rc.rs]: https://doc.rust-lang.org/stable/std/rc/struct.Rc.html
[Rc::clone]: https://doc.rust-lang.org/stable/std/rc/struct.Rc.html#method.clone

```rust
#![allow(dead_code, unused_variables)]

use std::rc::Rc;

struct Point {
    x: i32,
    y: i32,
}

impl Drop for Point {
    fn drop(&mut self) {
        println!("Point dropped!");
    }
}

fn main() {
    let a = Rc::new(Point { x: 12, y: 34 });
    let b = Rc::clone(&a); // share with b
    println!("a = {}, {}", a.x, a.y); // okay to use a
    println!("b = {}, {}", b.x, b.y);
}

// prints:
// a = 12, 34
// b = 12, 34
// Point dropped!
```

예제에서 확인할 점은 다음과 같습니다.

- `Point`는 `Drop`의 `drop` 메서드를 구현해 인스턴스가 드롭될 때
  메시지를 출력합니다.
- `main`에서 만든 점을 `Rc`가 감싸므로 점의 소유자는 `a`가 아닌
  스마트 포인터입니다.
- `b`는 스마트 포인터의 복제본을 받아 참조 횟수가 2가 됩니다.
  앞 예제와 달리 `a`와 `b`가 서로 다른 스마트 포인터 복제본을
  소유하므로 둘 다 계속 사용할 수 있습니다.
- `main` 끝에서 `a`와 `b`가 범위를 벗어나면 각각 드롭됩니다.
  `Rc`의 `Drop` 구현은 참조 횟수를 줄이고 0이 되면 소유한
  점도 드롭합니다. 이때 `Point`의 `Drop` 구현이 메시지를 한 번
  출력합니다. 점 하나를 만들어 공유한 뒤 한 번 드롭했다는 뜻입니다.

`Rc`는 스레드 안전하지 않습니다. 여러 스레드에서 소유권을 공유할
때는 Rust 표준 라이브러리의 [`Arc`][arc.rs]를 사용합니다.
Rust는 스레드 사이에서 `Rc`를 사용하지 못하도록 검사합니다.

[arc.rs]: https://doc.rust-lang.org/std/sync/struct.Arc.html

.NET에서는 C#의 `enum`, `struct` 같은 값 타입이 스택에 놓이고
`interface`, `record class`, `class` 같은 참조 타입은 힙에
할당된다고 설명하는 경우가 많습니다. Rust에서는 `enum`과 `struct`
중 무엇을 쓰는지가 메모리 위치를 결정하지 않습니다. 기본적으로
스택에 놓이지만 .NET에서 값 타입을 박싱해 힙에 복사하듯 Rust에서는
[`Box`][box.rs]로 감싸 힙에 할당할 수 있습니다.

[box.rs]: https://doc.rust-lang.org/std/boxed/struct.Box.html

```rust
let stack_point = Point { x: 12, y: 34 };
let heap_point = Box::new(Point { x: 12, y: 34 });
```

`Box`도 `Rc`, `Arc`처럼 스마트 포인터이지만 뒤에 있는 인스턴스를
독점적으로 소유합니다. 이 스마트 포인터들은 타입 인수 `T`의
인스턴스를 힙에 할당합니다.

C#의 `new` 키워드는 타입의 인스턴스를 만듭니다. 앞 예제의
`Box::new`, `Rc::new`도 비슷한 목적처럼 보일 수 있지만 Rust에서
`new`는 특별한 언어 구문이 아닙니다. 팩터리 함수에 관례적으로
붙이는 이름입니다. 이런 함수는 Rust에서 정적 메서드에 해당하는
타입의 _연관 함수_라고 부릅니다.
