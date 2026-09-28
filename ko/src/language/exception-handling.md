# 예외 처리

.NET에서 예외는 [`System.Exception`][net-system-exception] 클래스를
상속한 타입입니다. 코드 실행 중 문제가 생기면 예외를 던집니다.
프로그램이 예외를 처리하거나 종료할 때까지 예외는 호출 스택을 따라
전파됩니다.

Rust에는 예외가 없습니다. 대신 _복구 가능한_ 오류와 _복구할 수 없는_
오류를 구분합니다. 복구 가능한 오류는 문제를 보고한 뒤 프로그램을
계속 실행할 수 있는 경우를 뜻합니다. 이런 오류가 발생할 수 있는
연산은 [`Result<T, E>`][rust-result] 타입을 반환합니다. 여기서
`E`는 오류 변형의 타입입니다. 복구할 수 없는 오류를 만나면
[`panic!`][panic] 매크로가 실행을 중단합니다. 복구할 수 없는 오류는
항상 버그의 징후입니다.

## 사용자 정의 오류 타입

.NET의 사용자 정의 예외는 `Exception` 클래스를 상속합니다.
[사용자 정의 예외 작성 방법][net-user-defined-exceptions] 문서에는
다음 예제가 있습니다.

```csharp
public class EmployeeListNotFoundException : Exception
{
    public EmployeeListNotFoundException() { }

    public EmployeeListNotFoundException(string message)
        : base(message) { }

    public EmployeeListNotFoundException(string message, Exception inner)
        : base(message, inner) { }
}
```

Rust에서는 [`Error`][rust-std-error] 트레이트를 구현해 오류 값에
일반적으로 기대하는 기능을 제공할 수 있습니다. 최소한의 사용자
정의 오류 구현은 다음과 같습니다.

```rust
#[derive(Debug)]
pub struct EmployeeListNotFound;

impl std::fmt::Display for EmployeeListNotFound {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        f.write_str("Could not find employee list.")
    }
}

impl std::error::Error for EmployeeListNotFound {}
```

.NET의 `Exception.InnerException` 속성에 대응하는 기능은 Rust의
`Error::source()` 메서드입니다. 다만 이 메서드를 반드시 구현할
필요는 없습니다. 기본 일괄 구현은 `None`을 반환합니다.

> C#과 달리 Rust의 오류 타입이 반드시 `std::error::Error`를
> 구현해야 하는 것은 아닙니다. 이 트레이트 없이도 `Result`에서
> 사용할 수 있습니다. 공개 API의 오류 타입이라면 구현하는 관례가
> 있습니다.

## 오류 발생시키기

C#에서는 예외 인스턴스를 던져 예외를 발생시킵니다.

```csharp
void ThrowIfNegative(int value)
{
    if (value < 0)
    {
        throw new ArgumentOutOfRangeException(nameof(value));
    }
}
```

Rust에서는 복구 가능한 오류가 발생하면 메서드에서 `Ok` 또는
`Err` 변형을 반환합니다.

```rust
fn error_if_negative(value: i32) -> Result<(), &'static str> {
    if value < 0 {
        Err("Specified argument was out of the range of valid values. (Parameter 'value')")
    } else {
        Ok(())
    }
}
```

[`panic!`][panic] 매크로는 복구할 수 없는 오류를 발생시킵니다.

```rust
fn panic_if_negative(value: i32) {
    if value < 0 {
        panic!("Specified argument was out of the range of valid values. (Parameter 'value')")
    }
}
```

## 오류 전파

.NET에서는 예외를 처리하거나 프로그램이 종료될 때까지 예외가
호출 스택을 따라 전파됩니다. Rust에서 복구할 수 없는 오류도
비슷하게 동작하지만 이를 처리하는 일은 드뭅니다.

복구 가능한 오류는 명시적으로 전파하고 처리합니다. Rust의 함수나
메서드 시그니처는 이런 오류가 발생할 수 있음을 나타냅니다. C#에서는
예외를 잡아 오류 발생 여부에 따라 다음 동작을 선택할 수 있습니다.

```csharp
void Write()
{
    try
    {
        File.WriteAllText("file.txt", "content");
    }
    catch (IOException)
    {
        Console.WriteLine("Writing to file failed.");
    }
}
```

Rust에서는 대략 다음 코드에 대응합니다.

```rust
fn write() {
    match std::fs::write("file.txt", b"content")
    {
        Ok(_) => {}
        Err(_) => println!("Writing to file failed."),
    };
}
```

복구 가능한 오류를 직접 처리하지 않고 전파하기만 하는 경우도 많습니다.
이때 메서드 시그니처의 오류 타입이 전파할 오류와 호환되어야 합니다.
[`?` 연산자][question-mark-operator]를 사용하면 간결하게 전파할 수
있습니다.

```rust
fn write() -> Result<(), std::io::Error> {
    std::fs::write("file.txt", b"content")?;
    Ok(())
}
```

`?` 연산자로 오류를 전파하려면 [오류 전파의 간단한 방법][propagating-errors-rust-book]
절에서 설명하는 것처럼 오류 구현이 서로 _호환_되어야 합니다. 가장
범용적인 호환 오류 타입은 `Box<dyn Error>` [트레이트 객체]입니다.

## 스택 추적

.NET에서 처리되지 않은 예외를 던지면 런타임이 스택 추적을 출력해
문제의 맥락을 파악하도록 돕습니다.

Rust에서 복구할 수 없는 오류는 [`panic!`의 역추적][panic-backtrace]으로
비슷한 정보를 얻을 수 있습니다.

원문은 안정화된 Rust의 복구 가능한 오류에서 역추적을 지원하지
않는다고 설명합니다. 실험 기능의 [`provide` 메서드][provide method]를
사용하는 경우를 별도로 언급합니다.

[net-system-exception]: https://learn.microsoft.com/en-us/dotnet/api/system.exception?view=net-6.0
[rust-result]: https://doc.rust-lang.org/std/result/enum.Result.html
[panic-backtrace]: https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html#using-a-panic-backtrace
[net-user-defined-exceptions]: https://learn.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions
[rust-std-error]: https://doc.rust-lang.org/std/error/trait.Error.html
[provide method]: https://doc.rust-lang.org/std/error/trait.Error.html#method.provide
[question-mark-operator]: https://doc.rust-lang.org/std/result/index.html#the-question-mark-operator-
[panic]: https://doc.rust-lang.org/std/macro.panic.html
[propagating-errors-rust-book]: https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
[트레이트 객체]: https://doc.rust-lang.org/reference/types/trait-object.html
