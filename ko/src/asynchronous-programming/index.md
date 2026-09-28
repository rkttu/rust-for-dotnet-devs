# 비동기 프로그래밍

.NET과 Rust는 사용 방식이 비슷한 비동기 프로그래밍 모델을 지원합니다. 먼저 C# 비동기 코드의 기본 형태를 살펴보겠습니다.

```csharp
async Task<string> PrintDelayed(string message, CancellationToken cancellationToken)
{
    await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    return $"Message: {message}";
}
```

Rust 코드도 비슷하게 구성합니다. 다음 예제는 `sleep`을 구현하기 위해 [async-std]를 사용합니다.

```rust
use std::time::Duration;
use async_std::task::sleep;

async fn format_delayed(message: &str) -> String {
    sleep(Duration::from_secs(1)).await;
    format!("Message: {}", message)
}
```

1. Rust의 [`async`][async.rs] 키워드는 코드 블록을 [`Future`][future.rs] 트레이트를 구현하는 상태 기계로 바꿉니다. C# 컴파일러가 `async` 코드를 상태 기계로 바꾸는 방식과 비슷합니다. 두 언어 모두 이 방식으로 비동기 코드를 순차적인 형태로 작성할 수 있습니다.
2. Rust와 C# 모두 비동기 함수나 메서드 앞에 `async`를 붙이지만 반환 타입 표기는 다릅니다. C# 비동기 메서드는 `Task<T>`나 `ValueTask<T>`처럼 실제 반환 타입을 명시합니다. Rust 비동기 함수는 `String` 같은 내부 결과 타입을 명시합니다. 함수가 반환하는 값은 항상 `Future`를 구현하는 타입이기 때문입니다.
3. `await`의 위치도 다릅니다. C#에서는 표현식 앞에 `await`를 붙이고 Rust에서는 표현식 뒤에 `.await`를 붙입니다. Rust의 `.await`는 메서드가 아니지만 메서드 호출을 이어 쓰는 구문과 잘 어울립니다.

관련 내용은 [Rust 비동기 프로그래밍 문서][Asynchronous programming in Rust]에서 확인할 수 있습니다.

[async-std]: https://docs.rs/async-std/latest/async_std/
[async.rs]: https://doc.rust-lang.org/std/keyword.async.html
[future.rs]: https://doc.rust-lang.org/std/future/trait.Future.html
[Asynchronous programming in Rust]: https://rust-lang.github.io/async-book/

## 작업 실행

다음 C# 예제에서는 `PrintDelayed`의 반환 작업을 `await`하지 않아도 메서드가 실행됩니다.

```csharp
var cancellationToken = CancellationToken.None;
PrintDelayed("message", cancellationToken); // 1초 뒤 "message"를 출력합니다.
await Task.Delay(TimeSpan.FromSeconds(2), cancellationToken);

async Task PrintDelayed(string message, CancellationToken cancellationToken)
{
    await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    Console.WriteLine(message);
}
```

Rust에서 같은 방식으로 함수를 호출하면 아무것도 출력하지 않습니다.

```rust
use async_std::task::sleep;
use std::time::Duration;

#[tokio::main] // 비동기 main 함수를 실행하기 위해 사용합니다.
async fn main() {
    print_delayed("message"); // 아무것도 출력하지 않습니다.
    sleep(Duration::from_secs(2)).await;
}

async fn print_delayed(message: &str) {
    sleep(Duration::from_secs(1)).await;
    println!("{}", message);
}
```

Rust의 퓨처는 지연 실행되기 때문입니다. 실행하기 전까지 아무 작업도 수행하지 않습니다. `Future`를 실행하는 일반적인 방법은 `.await`를 사용하는 것입니다. `.await`는 퓨처를 완료할 때까지 실행하려고 시도합니다. 퓨처가 더 진행할 수 없으면 현재 스레드의 제어권을 양보합니다. 다시 진행할 수 있을 때 실행기가 퓨처를 선택해 실행을 재개하고 `.await`가 완료됩니다. 자세한 내용은 [`async/.await`][async-await.rs] 문서를 참고할 수 있습니다.

다른 `async` 함수 안에서는 퓨처를 기다릴 수 있지만 Rust의 표준 `main` 함수는 그대로 [`async`로 선언할 수 없습니다][error-E0752]. Rust 자체가 비동기 코드를 실행하는 런타임을 제공하지 않기 때문입니다. 비동기 코드를 실행하려면 [비동기 런타임][async runtimes]을 사용합니다. [Tokio][tokio.rs]는 널리 쓰이는 비동기 런타임 가운데 하나입니다. 위 예제의 [`tokio::main`][tokio-main.rs] 특성 매크로는 비동기 `main` 함수를 진입점으로 지정하고 런타임을 구성합니다.

[tokio.rs]: https://crates.io/crates/tokio
[tokio-main.rs]: https://docs.rs/tokio/latest/tokio/attr.main.html
[async-await.rs]: https://rust-lang.github.io/async-book/03_async_await/01_chapter.html#asyncawait
[error-E0752]: https://doc.rust-lang.org/error-index.html#E0752
[async runtimes]: https://rust-lang.github.io/async-book/08_ecosystem/00_chapter.html#async-runtimes

## 작업 취소

앞선 C# 예제에서는 .NET의 일반적인 방식에 따라 비동기 메서드에 `CancellationToken`을 전달했습니다. `CancellationToken`으로 비동기 작업의 취소를 알릴 수 있습니다.

Rust의 퓨처는 폴링할 때만 진행하므로 취소 방식이 다릅니다. `Future`를 해제하면 더 이상 진행하지 않습니다. 대기 중인 비동기 작업에서 중단된 지점까지 생성한 값도 함께 해제됩니다. 따라서 대부분의 Rust 비동기 함수는 취소 신호를 받는 매개변수를 사용하지 않습니다. 퓨처를 해제하는 동작을 취소라고 부르기도 합니다.

퓨처의 소유권을 해제하는 방식이 적합하지 않은 경우에는 [`tokio_util::sync::CancellationToken`][cancellation-token.rs]으로 .NET의 `CancellationToken`과 비슷하게 취소를 알리고 처리할 수 있습니다.

[cancellation-token.rs]: https://docs.rs/tokio-util/latest/tokio_util/sync/struct.CancellationToken.html

## 여러 작업 실행

.NET에서는 `Task.WhenAny`와 `Task.WhenAll`을 사용해 여러 작업을 처리합니다.

`Task.WhenAny`는 작업 하나가 완료되면 완료됩니다. Tokio의 [`tokio::select!`][tokio-select] 매크로도 동시에 진행되는 여러 분기 중 하나가 완료되기를 기다릴 때 사용할 수 있습니다. 먼저 C# 예제를 살펴보겠습니다.

```csharp
var cancellationToken = CancellationToken.None;

var result =
    await Task.WhenAny(Delay(TimeSpan.FromSeconds(2), cancellationToken),
                       Delay(TimeSpan.FromSeconds(1), cancellationToken));

Console.WriteLine(result.Result); // Waited 1 second(s).

async Task<string> Delay(TimeSpan delay, CancellationToken cancellationToken)
{
    await Task.Delay(delay, cancellationToken);
    return $"Waited {delay.TotalSeconds} second(s).";
}
```

Rust에서는 다음처럼 작성합니다.

```rust
use std::time::Duration;
use tokio::{select, time::sleep};

#[tokio::main]
async fn main() {
    let result = select! {
        result = delay(Duration::from_secs(2)) => result,
        result = delay(Duration::from_secs(1)) => result,
    };

    println!("{}", result); // Waited 1 second(s).
}

async fn delay(delay: Duration) -> String {
    sleep(delay).await;
    format!("Waited {} second(s).", delay.as_secs())
}
```

두 예제는 취소 동작이 다릅니다. `tokio::select!`는 선택되지 않은 나머지 분기를 취소합니다. `Task.WhenAny`를 사용할 때는 아직 실행 중인 작업을 취소할지 개발자가 결정합니다.

`Task.WhenAll`에 대응하는 기능으로는 [`tokio::join!`][tokio-join]을 사용할 수 있습니다.

[tokio-select]: https://docs.rs/tokio/latest/tokio/macro.select.html
[tokio-join]: https://docs.rs/tokio/latest/tokio/macro.join.html

## 여러 소비자

.NET의 `Task`는 여러 소비자가 기다릴 수 있습니다. 작업이 완료되거나 실패하면 모두 결과를 전달받습니다. Rust의 `Future`는 일반적으로 복제하거나 복사할 수 없으며 `.await`하면 소유권이 이동합니다. `futures::FutureExt::shared` 확장 메서드는 여러 소비자에게 전달할 수 있는 복제 가능한 퓨처 핸들을 만듭니다.

```rust
use futures::FutureExt;
use std::time::Duration;
use tokio::{select, time::sleep, signal};
use tokio_util::sync::CancellationToken;

#[tokio::main]
async fn main() {
    let token = CancellationToken::new();
    let child_token = token.child_token();

    let bg_operation = background_operation(child_token);

    let bg_operation_done = bg_operation.shared();
    let bg_operation_final = bg_operation_done.clone();

    select! {
        _ = bg_operation_done => {},
        _ = signal::ctrl_c() => {
            token.cancel();
        },
    }

    bg_operation_final.await;
}

async fn background_operation(cancellation_token: CancellationToken) {
    select! {
        _ = sleep(Duration::from_secs(2)) => println!("Background operation completed."),
        _ = cancellation_token.cancelled() => println!("Background operation cancelled."),
    }
}
```

## 비동기 반복

.NET에는 [`IAsyncEnumerable<T>`][async-enumerable.net]와 [`IAsyncEnumerator<T>`][async-enumerator.net]가 있습니다. 원문 작성 당시 Rust 표준 라이브러리에는 이에 대응하는 비동기 반복 API가 없었습니다. 대신 [`futures`의 `Stream` 트레이트][futures-stream.rs]가 비슷한 기능을 제공합니다. 개념과 사용법은 [Rust 비동기 책의 스트림 장][stream.rs]에서도 다룹니다.

C#에서는 동기 반복자를 작성할 때와 비슷한 구문으로 비동기 반복자를 작성합니다.

```csharp
await foreach (int item in RangeAsync(10, 3).WithCancellation(CancellationToken.None))
    Console.Write(item + " "); // "10 11 12"를 출력합니다.

async IAsyncEnumerable<int> RangeAsync(int start, int count)
{
    for (int i = 0; i < count; i++)
    {
        await Task.Delay(TimeSpan.FromSeconds(i));
        yield return start + i;
    }
}
```

Rust에는 `futures::channel::mpsc`처럼 `Stream`을 구현하는 여러 타입이 있습니다. C#에 가까운 구문을 원한다면 [`async-stream`][tokio-async-stream]이 제공하는 매크로로 스트림을 간결하게 생성할 수 있습니다.

```rust
use async_stream::stream;
use futures_core::stream::Stream;
use futures_util::{pin_mut, stream::StreamExt};
use std::{
    io::{stdout, Write},
    time::Duration,
};
use tokio::time::sleep;

#[tokio::main]
async fn main() {
    let stream = range(10, 3);
    pin_mut!(stream); // 반복에 필요합니다.
    while let Some(result) = stream.next().await {
        print!("{} ", result); // "10 11 12"를 출력합니다.
        stdout().flush().unwrap();
    }
}

fn range(start: i32, count: i32) -> impl Stream<Item = i32> {
    stream! {
        for i in 0..count {
            sleep(Duration::from_secs(i as _)).await;
            yield start + i;
        }
    }
}
```

[async-enumerable.net]: https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iasyncenumerable-1
[async-enumerator.net]: https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.iasyncenumerator-1
[stream.rs]: https://rust-lang.github.io/async-book/05_streams/01_chapter.html
[futures-stream.rs]: https://docs.rs/futures/latest/futures/stream/trait.Stream.html
[tokio-async-stream]: https://github.com/tokio-rs/async-stream
