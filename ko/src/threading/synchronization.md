# 동기화

여러 스레드가 데이터를 공유한다면 손상을 막기 위해 읽기와 쓰기
접근을 동기화합니다. C#에서는 `lock` 키워드를 동기화 기본
구문으로 제공합니다. .NET의 `Monitor`를 예외 상황에도 안전하게
사용하도록 펼쳐지는 구문입니다.

```csharp
using System;
using System.Threading;

var dataLock = new object();
var data = 0;
var threads = new List<Thread>();

for (var i = 0; i < 10; i++)
{
    var thread = new Thread(() =>
    {
        for (var j = 0; j < 1000; j++)
        {
            lock (dataLock)
                data++;
        }
    });
    threads.Add(thread);
    thread.Start();
}

foreach (var thread in threads)
    thread.Join();

Console.WriteLine(data);
```

Rust에서는 `Mutex` 같은 동시성 자료구조를 명시적으로 사용합니다.

```rust
use std::thread;
use std::sync::{Arc, Mutex};

fn main() {
    let data = Arc::new(Mutex::new(0)); // (1)

    let mut threads = vec![];
    for _ in 0..10 {
        let data = Arc::clone(&data); // (2)
        let thread = thread::spawn(move || { // (3)
            for _ in 0..1000 {
                let mut data = data.lock().unwrap();
                *data += 1; // (4)
            }
        });
        threads.push(thread);
    }

    for thread in threads {
        thread.join().unwrap();
    }

    println!("{}", data.lock().unwrap());
}
```

예제에서 확인할 점은 다음과 같습니다.

- 여러 스레드가 `Mutex` 인스턴스와 그 안의 데이터를 함께 소유하므로
  `Arc`로 감쌉니다(1). `Arc`는 원자적 참조 횟수를 제공하며 복제할
  때 증가하고(2) 드롭할 때 감소합니다. 횟수가 0이 되면 뮤텍스와
  그 안의 데이터가 드롭됩니다. [메모리 관리] 장에서 자세히
  설명합니다.
- 각 스레드의 클로저는 복제된 참조(2)의 소유권을 받습니다(3).
- `*data += 1`(4)은 포인터 접근처럼 보이지만 안전하지 않은
  포인터 조작이 아닙니다. [뮤텍스 가드]가 감싼 데이터를
  갱신합니다.

C# 예제는 `lock` 문을 제거하면 스레드 안전하지 않은 코드가 될
수 있습니다. Rust 예제는 일부 코드를 제거하는 등 스레드 안전성을
해치는 변경을 하면 컴파일을 거부합니다. C#/.NET에서는 개발자가
동기화 자료구조를 주의해서 사용해야 하며 Rust에서는 컴파일러의
검사를 받을 수 있다는 차이를 보여 줍니다.

Rust 컴파일러가 이를 검사할 수 있는 이유는 자료구조에 `Sync`,
`Send`라는 특별한 [트레이트][인터페이스]가 적용되기 때문입니다.
[`Sync`][sync.rs]는 타입 인스턴스의 참조를 스레드 사이에서
공유해도 안전함을 뜻합니다. [`Send`][send.rs]는 인스턴스를
다른 스레드로 보내거나 이동해도 안전함을 뜻합니다. 자세한 내용은
Rust 책의 [두려움 없는 동시성] 장에서 확인할 수 있습니다.

[두려움 없는 동시성]: https://doc.rust-lang.org/book/ch16-00-concurrency.html
[메모리 관리]: ../memory-management/index.md
[뮤텍스 가드]: https://doc.rust-lang.org/stable/std/sync/struct.MutexGuard.html
[sync.rs]: https://doc.rust-lang.org/stable/std/marker/trait.Sync.html
[send.rs]: https://doc.rust-lang.org/stable/std/marker/trait.Send.html
[인터페이스]: ../language/custom-types/interfaces.md
