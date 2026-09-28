# 스레딩

Rust 표준 라이브러리는 스레딩, 동기화, 동시성을 지원합니다.
언어와 표준 라이브러리에 기본 기능이 있으며 crate는 더 많은 기능을
제공합니다. 이 안내서에서는 추가 crate를 다루지 않습니다.

다음 표는 .NET의 스레딩 타입과 메서드가 Rust에서 대략 무엇에
대응하는지 보여 줍니다.

| .NET               | Rust                      |
| ------------------ | ------------------------- |
| `Thread`           | `std::thread::thread`     |
| `Thread.Start`     | `std::thread::spawn`      |
| `Thread.Join`      | `std::thread::JoinHandle` |
| `Thread.Sleep`     | `std::thread::sleep`      |
| `ThreadPool`       | 해당 없음                 |
| `Mutex`            | `std::sync::Mutex`        |
| `Semaphore`        | 해당 없음                 |
| `Monitor`          | `std::sync::Mutex`        |
| `ReaderWriterLock` | `std::sync::RwLock`       |
| `AutoResetEvent`   | `std::sync::Condvar`      |
| `ManualResetEvent` | `std::sync::Condvar`      |
| `Barrier`          | `std::sync::Barrier`      |
| `CountdownEvent`   | `std::sync::Barrier`      |
| `Interlocked`      | `std::sync::atomic`       |
| `Volatile`         | `std::sync::atomic`       |
| `ThreadLocal`      | `std::thread_local`       |

C#/.NET과 Rust에서는 비슷한 방식으로 스레드를 시작하고 완료될
때까지 기다립니다. 다음 C# 프로그램은 스레드를 만들고 그 스레드가
표준 출력에 문장을 출력한 뒤 종료되기를 기다립니다.

```csharp
using System;
using System.Threading;

var thread = new Thread(() => Console.WriteLine("Hello from a thread!"));
thread.Start();
thread.Join(); // wait for thread to finish
```

Rust의 대응 코드는 다음과 같습니다.

```rust
use std::thread;

fn main() {
    let thread = thread::spawn(|| println!("Hello from a thread!"));
    thread.join().unwrap(); // wait for thread to finish
}
```

.NET에서는 스레드 객체의 생성과 초기화, 스레드 시작이 서로 다른
동작입니다. Rust의 `thread::spawn`은 두 동작을 함께 수행합니다.

.NET에서는 스레드에 인수로 데이터를 전달할 수 있습니다.

```csharp
#nullable enable

using System;
using System.Text;
using System.Threading;

var t = new Thread(obj =>
{
    var data = (StringBuilder)obj!;
    data.Append(" World!");
});

var data = new StringBuilder("Hello");
t.Start(data);
t.Join();

Console.WriteLine($"Phrase: {data}");
```

클로저를 사용하면 더 간결하고 현대적인 C# 코드로 작성할 수 있습니다.

```csharp
using System;
using System.Text;
using System.Threading;

var data = new StringBuilder("Hello");

var t = new Thread(obj => data.Append(" World!"));

t.Start();
t.Join();

Console.WriteLine($"Phrase: {data}");
```

Rust의 `thread::spawn`에는 같은 인수 전달 방식이 없습니다. 대신
클로저를 통해 스레드에 데이터를 전달합니다.

```rust
use std::thread;

fn main() {
    let data = String::from("Hello");
    let handle = thread::spawn(move || {
        let mut data = data;
        data.push_str(" World!");
        data
    });
    println!("Phrase: {}", handle.join().unwrap());
}
```

예제에서 확인할 점은 다음과 같습니다.

- `move` 키워드로 `data`의 소유권을 스레드의 클로저로 _이동_해야
  합니다. 그 뒤에는 `main`에서 원래 `data` 변수를 사용할 수
  없습니다. 계속 사용하려면 값 타입의 지원 범위에 따라 복사하거나
  복제합니다.

  Rust 1.63.0부터는 [범위 지정 스레드]로 정적 수명이 아닌 데이터와
  이동하지 않은 값의 참조도 스레드에서 사용할 수 있습니다. 데이터가
  스레드 종료까지 살아 있어야 하므로 범위가 끝나기 전에 스레드를
  강제로 합류시킵니다.

- Rust 스레드는 C#의 태스크처럼 값을 반환할 수 있으며 이 값은
  `join` 메서드의 반환값이 됩니다.
- C#에서도 Rust 예제처럼 클로저로 스레드에 데이터를 전달할 수
  있습니다. C#은 소유권을 직접 관리하지 않습니다. 참조가 모두
  사라지면 GC가 데이터의 메모리를 회수합니다.

[범위 지정 스레드]: https://doc.rust-lang.org/stable/std/thread/fn.scope.html
