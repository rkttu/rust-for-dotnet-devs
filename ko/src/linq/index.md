# LINQ

이 장에서는 시퀀스(`IEnumerable`/`IEnumerable<T>`)를 조회하거나 변환할 때 사용하는 LINQ를 다룹니다. 리스트, 집합, 딕셔너리 같은 컬렉션도 포함합니다.

## `IEnumerable<T>`

Rust에서 `IEnumerable<T>`에 대응하는 트레이트는 [`IntoIterator`][into-iter.rs]입니다. .NET에서 `IEnumerable<T>.GetEnumerator()`가 `IEnumerator<T>`를 반환하듯, Rust의 `IntoIterator::into_iter`는 [`Iterator`][iter.rs]를 반환합니다. 두 언어 모두 반복 가능한 컨테이너의 항목을 순회하는 간단한 구문을 제공합니다. C#에서는 `foreach`를 사용합니다.

```csharp
using System;
using System.Text;

var values = new[] { 1, 2, 3, 4, 5 };
var output = new StringBuilder();

foreach (var value in values)
{
    if (output.Length > 0)
        output.Append(", ");
    output.Append(value);
}

Console.Write(output); // 출력: 1, 2, 3, 4, 5
```

Rust에서는 `for`를 사용합니다.

```rust
use std::fmt::Write;

fn main() {
    let values = [1, 2, 3, 4, 5];
    let mut output = String::new();

    for value in values {
        if output.len() > 0 {
            output.push_str(", ");
        }
        // ! 쓰기 오류를 무시합니다.
        _ = write!(output, "{value}");
    }

    println!("{output}");  // 출력: 1, 2, 3, 4, 5
}
```

Rust의 `for` 반복문은 대략 다음 코드로 변환됩니다.

```rust
use std::fmt::Write;

fn main() {
    let values = [1, 2, 3, 4, 5];
    let mut output = String::new();

    let mut iter = values.into_iter();      // 반복자를 가져옵니다.
    while let Some(value) = iter.next() {   // 항목이 남아 있는 동안 반복합니다.
        if output.len() > 0 {
            output.push_str(", ");
        }
        _ = write!(output, "{value}");
    }

    println!("{output}");
}
```

Rust에서는 순회할 때도 소유권과 데이터 경합 방지 규칙이 적용됩니다. 배열을 순회하는 모습은 C#과 비슷해 보이지만 같은 컬렉션을 두 번 이상 순회하려면 소유권을 고려해야 합니다. 다음 예제는 정수 벡터를 두 번 순회하여 합계와 최댓값을 각각 출력하려고 합니다.

```rust
fn main() {
    let values = vec![1, 2, 3, 4, 5];

    // 모든 값을 더합니다.

    let mut sum = 0;
    for value in values {
        sum += value;
    }
    println!("sum = {sum}");

    // 최댓값을 구합니다.

    let mut max = None;
    for value in values {
        if let Some(some_max) = max { // max가 정의되어 있으면
            if value > some_max {     // value가 더 크면
                max = Some(value)     // 새 최댓값을 기록합니다.
            }
        } else {                      // 반복 시작 시 max는 정의되지 않았으므로
            max = Some(value)         // 첫 번째 값으로 설정합니다.
        }
    }
    println!("max = {max:?}");
}
```

하지만 이 코드는 컴파일되지 않습니다. `values`가 배열 대신 [`Vec<i32>`][vec.rs]로 선언되어 있기 때문입니다. 벡터는 .NET의 `List<T>`처럼 크기를 늘릴 수 있는 Rust의 컬렉션입니다. 첫 번째 반복문에서 벡터의 항목을 순회 변수 `value`로 옮기면서 벡터를 소비합니다. 각 `value`는 반복이 끝날 때 범위를 벗어나 해제됩니다. 항목이 힙 메모리를 소유한다면 해당 메모리도 함께 해제됩니다. 이를 해결하려면 `for` 반복문에서 `&values`를 사용해 항목에 대한 공유 참조를 순회합니다. 그러면 `value`가 항목의 소유권을 가져오지 않습니다.

[vec.rs]: https://doc.rust-lang.org/stable/std/vec/struct.Vec.html

다음은 각 `for` 반복문의 `values`를 `&values`로 바꾼 코드입니다. 이제 컴파일할 수 있습니다.

```rust
fn main() {
    let values = vec![1, 2, 3, 4, 5];

    // 모든 값을 더합니다.

    let mut sum = 0;
    for value in &values {
        sum += value;
    }
    println!("sum = {sum}");

    // 최댓값을 구합니다.

    let mut max = None;
    for value in &values {
        if let Some(some_max) = max { // max가 정의되어 있으면
            if value > some_max {     // value가 더 크면
                max = Some(value)     // 새 최댓값을 기록합니다.
            }
        } else {                      // 반복 시작 시 max는 정의되지 않았으므로
            max = Some(value)         // 첫 번째 값으로 설정합니다.
        }
    }
    println!("max = {max:?}");
}
```

벡터 대신 배열을 사용해도 소유권 이전과 값의 해제를 관찰할 수 있습니다. 다음 예제는 정수를 감싼 `Int` 구조체의 배열을 순회하며 합계를 계산합니다.

```rust
#[derive(Debug)]
struct Int(i32);

impl Drop for Int {
    fn drop(&mut self) {
        println!("{:?} dropped", self)
    }
}

fn main() {
    let values = [Int(1), Int(2), Int(3), Int(4), Int(5)];
    let mut sum = 0;

    for value in values {
        println!("value = {:?}", value);
        sum += value.0;
    }

    println!("sum = {sum}");
}
```

`Int`는 `Drop`을 구현하여 인스턴스가 해제될 때 메시지를 출력합니다. 위 코드를 실행하면 다음 결과를 얻습니다.

    value = Int(1)
    Int(1) dropped
    value = Int(2)
    Int(2) dropped
    value = Int(3)
    Int(3) dropped
    value = Int(4)
    Int(4) dropped
    value = Int(5)
    Int(5) dropped
    sum = 15

반복문이 각 값을 가져오고 해제한 뒤 합계를 출력합니다. 반복문에서 `values`를 `&values`로 바꾸면 다음과 같습니다.

```rust
for value in &values {
    // ...
}
```

출력 순서도 달라집니다.

    value = Int(1)
    value = Int(2)
    value = Int(3)
    value = Int(4)
    value = Int(5)
    sum = 15
    Int(1) dropped
    Int(2) dropped
    Int(3) dropped
    Int(4) dropped
    Int(5) dropped

이번에는 반복 변수가 항목의 소유권을 가져오지 않으므로 순회 중에는 값이 해제되지 않습니다. 반복이 끝나고 합계를 출력한 뒤 `main`의 끝에서 `values` 배열이 범위를 벗어나면 모든 `Int` 인스턴스가 해제됩니다.

이 예제처럼 Rust와 C#의 반복 구문과 추상화는 비슷하지만 소유권에 따른 차이가 있습니다. 이 차이로 인해 Rust 컴파일러가 일부 반복 코드를 거부할 수 있습니다.

관련 내용은 [Iterator][iter-mod]와 [참조를 통한 순회][iterating by reference] 문서를 참고할 수 있습니다.

[into-iter.rs]: https://doc.rust-lang.org/std/iter/trait.IntoIterator.html
[iter.rs]: https://doc.rust-lang.org/core/iter/trait.Iterator.html
[iter-mod]: https://doc.rust-lang.org/std/iter/index.html
[iterating by reference]: https://doc.rust-lang.org/std/iter/index.html#iterating-by-reference

## 연산자

LINQ의 _연산자_는 연결해서 사용할 수 있는 C# 확장 메서드입니다. 일반적으로 데이터 원본에 대한 조회를 구성합니다. C#은 `from`, `where`, `select`, `join` 같은 절을 사용하는 SQL 형태의 _쿼리 구문_도 제공합니다. 쿼리 구문은 메서드 연결을 대신하거나 함께 사용할 수 있습니다. 많은 명령형 반복문을 표현력과 조합성이 높은 LINQ 쿼리로 바꿀 수 있습니다.

Rust에는 C#의 쿼리 구문에 대응하는 구문이 없습니다. 대신 반복 가능한 타입에 적용하는 메서드, 즉 _[어댑터][adapters]_가 있으며 C#의 메서드 연결과 비슷하게 사용할 수 있습니다. C#에서는 명령형 반복문을 LINQ로 바꿀 때 표현력과 조합성이 좋아지는 대신 성능을 고려할 수 있습니다. 계산량이 많은 명령형 반복문은 JIT 최적화를 활용하고 간접 호출 횟수를 줄일 수 있어 보통 더 빠릅니다. Rust에서는 반복자 메서드를 연결하는 코드와 직접 작성한 명령형 반복문 사이에 이러한 성능 차이가 일반적으로 발생하지 않습니다. 따라서 반복자 메서드를 연결하는 코드를 자주 볼 수 있습니다.

다음 표는 자주 쓰는 LINQ 메서드와 Rust의 대략적인 대응 메서드를 보여 줍니다.

| .NET              | Rust         | 비고 |
| ----------------- | ------------ | ---- |
| `Aggregate`       | `reduce`     | 주 1 |
| `Aggregate`       | `fold`       | 주 1 |
| `All`             | `all`        |      |
| `Any`             | `any`        |      |
| `Concat`          | `chain`      |      |
| `Count`           | `count`      |      |
| `ElementAt`       | `nth`        |      |
| `GroupBy`         | -            |      |
| `Last`            | `last`       |      |
| `Max`             | `max`        |      |
| `Max`             | `max_by`     |      |
| `MaxBy`           | `max_by_key` |      |
| `Min`             | `min`        |      |
| `Min`             | `min_by`     |      |
| `MinBy`           | `min_by_key` |      |
| `Reverse`         | `rev`        |      |
| `Select`          | `map`        |      |
| `Select`          | `enumerate`  |      |
| `SelectMany`      | `flat_map`   |      |
| `SelectMany`      | `flatten`    |      |
| `SequenceEqual`   | `eq`         |      |
| `Single`          | -            | 주 3 |
| `SingleOrDefault` | -            | 주 3 |
| `Skip`            | `skip`       |      |
| `SkipWhile`       | `skip_while` |      |
| `Sum`             | `sum`        |      |
| `Take`            | `take`       |      |
| `TakeWhile`       | `take_while` |      |
| `ToArray`         | `collect`    | 주 2 |
| `ToDictionary`    | `collect`    | 주 2 |
| `ToList`          | `collect`    | 주 2 |
| `Where`           | `filter`     |      |
| `Zip`             | `zip`        |      |

1. 초기값을 받지 않는 `Aggregate` 오버로드는 `reduce`에, 초기값을 받는 오버로드는 `fold`에 대응합니다.
2. Rust의 [`collect`][collect.rs]는 일반적으로 반복자로부터 값을 생성할 수 있는 모든 타입, 즉 [`FromIterator`를 구현한 타입][FromIter.rs]에 사용할 수 있습니다. `collect`는 대상 타입이 필요합니다. 컴파일러가 이를 추론하지 못하면 `collect::<Vec<_>>()`처럼 터보피시(`::<>`) 구문으로 타입을 지정합니다. 이 때문에 표에서는 열거 가능한 값을 컬렉션으로 변환하는 여러 LINQ 메서드에 `collect`를 대응시켰습니다.
3. [`Single`][single.net]과 [`SingleOrDefault`][single-or-default.net]는 결과가 둘 이상인지 검사합니다. Rust의 [`find`][find.rs]는 첫 번째 일치 항목만 반환하며 유일성은 검사하지 않습니다. 원문 표의 `find`와 `try_find` 대응 관계는 이 차이를 반영하지 못하므로 직접 대응 메서드를 표시하지 않았습니다.

[FromIter.rs]: https://doc.rust-lang.org/stable/std/iter/trait.FromIterator.html
[single.net]: https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.single
[single-or-default.net]: https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.singleordefault
[find.rs]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.find

다음은 C#에서 시퀀스를 변환하는 예제입니다.

```csharp
var result =
    Enumerable.Range(0, 10)
              .Where(x => x % 2 == 0)
              .SelectMany(x => Enumerable.Range(0, x))
              .Aggregate(0, (acc, x) => acc + x);

Console.WriteLine(result); // 50
```

Rust에서도 비슷하게 작성할 수 있습니다.

```rust
let result = (0..10)
    .filter(|x| x % 2 == 0)
    .flat_map(|x| (0..x))
    .fold(0, |acc, x| acc + x);

println!("{result}"); // 50
```

[adapters]: https://doc.rust-lang.org/std/iter/index.html#adapters
[collect.rs]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect

## 지연 실행

LINQ의 많은 연산자는 결과가 실제로 필요할 때 작업을 수행하도록 설계되었습니다. 이를 통해 여러 작업을 연결할 때 불필요한 계산을 피할 수 있습니다. 예를 들어 LINQ 연산자가 `IEnumerable<T>`를 반환해도 순회를 시작하기 전에는 `T` 항목을 생성하거나 계산하지 않을 수 있습니다. 이를 _지연 실행_이라고 합니다. 순회가 시작될 때 모든 항목을 만드는 대신 각 항목에 도달할 때마다 계산하면 결과를 _스트리밍_한다고 합니다.

Rust 반복자도 같은 [_지연 평가_][iter-laziness]와 스트리밍 개념을 지원합니다.

[iter-laziness]: https://doc.rust-lang.org/std/iter/index.html#laziness

두 언어 모두 이러한 특성으로 _무한 시퀀스_를 표현할 수 있습니다. 원본 시퀀스는 끝이 없지만 개발자가 필요한 만큼만 소비합니다. 다음은 C# 예제입니다.

```csharp
foreach (var x in InfiniteRange().Take(5))
    Console.Write($"{x} "); // "0 1 2 3 4"를 출력합니다.

IEnumerable<int> InfiniteRange()
{
    for (var i = 0; ; ++i)
        yield return i;
}
```

Rust에서는 무한 범위로 같은 개념을 표현합니다.

```rust
// 원문 작성 당시 Rust의 제너레이터와 yield는 불안정 기능이므로
// 이 예제에서는 대신 Range를 사용합니다.
// https://doc.rust-lang.org/std/ops/struct.Range.html

for value in (0..).take(5) {
    print!("{value} "); // "0 1 2 3 4"를 출력합니다.
}
```

## 반복자 메서드(`yield`)

C#의 `yield` 키워드를 사용하면 _반복자 메서드_를 간단히 작성할 수 있습니다. 반복자 메서드는 `IEnumerable<T>` 또는 `IEnumerator<T>`를 반환합니다. 컴파일러가 메서드 본문을 반환 타입의 구체적인 구현으로 변환하므로 개발자가 매번 클래스를 직접 작성하지 않아도 됩니다. 원문 작성 당시 Rust의 대응 기능인 [_코루틴_][coroutines.rs]은 불안정 기능으로 분류되었습니다.

[coroutines.rs]: https://doc.rust-lang.org/unstable-book/language-features/coroutines.html
