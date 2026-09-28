# 구조화된 타입

다음 표는 .NET에서 자주 쓰는 객체 및 컬렉션 타입과 Rust의 대응 타입을
보여 줍니다.

| C#           | Rust      |
| ------------ | --------- |
| `Array`      | `Array`   |
| `List`       | `Vec`     |
| `Tuple`      | `Tuple`   |
| `Dictionary` | `HashMap` |

## 배열

Rust도 .NET처럼 길이가 고정된 배열을 지원합니다. 먼저 C# 예제입니다.

```csharp
int[] someArray = new int[2] { 1, 2 };
```

Rust에서는 다음과 같이 작성합니다.

```rust
let someArray: [i32; 2] = [1,2];
```

## 목록

Rust의 `Vec<T>`는 C#의 `List<T>`에 대응합니다. 배열을 벡터로
변환하거나 벡터를 배열로 변환할 수 있습니다. 먼저 C# 예제입니다.

```csharp
var something = new List<string>
{
    "a",
    "b"
};

something.Add("c");
```

Rust 예제는 다음과 같습니다.

```rust
let mut something = vec![
    "a".to_owned(),
    "b".to_owned()
];

something.push("c".to_owned());
```

## 튜플

먼저 C#의 튜플 예제입니다.

```csharp
var something = (1, 2)
Console.WriteLine($"a = {something.Item1} b = {something.Item2}");
```

Rust의 튜플은 다음과 같습니다.

```rust
let something = (1, 2);
println!("a = {} b = {}", something.0, something.1);

// 분해를 지원합니다.
let (a, b) = something;
println!("a = {} b = {}", a, b);
```

> Rust는 C#과 달리 튜플 요소에 이름을 붙일 수 없습니다. 인덱스로
> 접근하거나 튜플을 분해해서 요소를 사용할 수 있습니다.

## 딕셔너리

Rust의 `HashMap<K, V>`는 C#의 `Dictionary<TKey, TValue>`에
대응합니다. 먼저 C# 예제입니다.

```csharp
var something = new Dictionary<string, string>
{
    { "Foo", "Bar" },
    { "Baz", "Qux" }
};

something.Add("hi", "there");
```

Rust 예제는 다음과 같습니다.

```rust
let mut something = HashMap::from([
    ("Foo".to_owned(), "Bar".to_owned()),
    ("Baz".to_owned(), "Qux".to_owned())
]);

something.insert("hi".to_owned(), "there".to_owned());
```

관련 자료:

- [Rust 표준 라이브러리의 컬렉션](https://doc.rust-lang.org/std/collections/index.html)
