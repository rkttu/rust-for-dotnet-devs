# 생산자와 소비자

생산자와 소비자 패턴은 스레드 사이에서 작업을 분배할 때 자주
사용합니다. 생산자 스레드가 소비자 스레드에 데이터를 전달하므로
데이터를 공유하거나 잠글 필요가 없습니다. .NET은 이 패턴을 폭넓게
지원합니다. 기본적인 예로 `System.Collections.Concurrent`의
`BlockingCollection`을 사용하는 C# 코드를 살펴보겠습니다.

```csharp
using System;
using System.Threading;
using System.Collections.Concurrent;

var messages = new BlockingCollection<string>();
var producer = new Thread(() =>
{
    for (var n = 1; i < 10; i++)
        messages.Add($"Message #{n}");
    messages.CompleteAdding();
});

producer.Start();

// main thread is the consumer here
foreach (var message in messages.GetConsumingEnumerable())
    Console.WriteLine(message);

producer.Join();
```

Rust에서는 _채널_로 같은 동작을 구현할 수 있습니다. 표준
라이브러리의 `mpsc::channel`은 생산자 여럿과 소비자 하나를
지원합니다. 위 C# 예제를 Rust로 옮기면 대략 다음과 같습니다.

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    let producer = thread::spawn(move || {
        for n in 1..10 {
            tx.send(format!("Message #{}", n)).unwrap();
        }
    });

    // main thread is the consumer here
    for received in rx {
        println!("{}", received);
    }

    producer.join().unwrap();
}
```

.NET에도 `System.Threading.Channels` 네임스페이스에 채널이
있습니다. 주로 `async`와 `await`를 사용하는 태스크 및 비동기
프로그래밍을 위해 설계되었습니다. Rust에서 비동기 코드에 적합한
채널은 [Tokio 런타임][tokio-channels]이 제공합니다.

[tokio-channels]: https://tokio.rs/tokio/tutorial/channels
