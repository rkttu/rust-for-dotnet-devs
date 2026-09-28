# 연산자 오버로딩

C#에서는 사용자 정의 타입이 _오버로드 가능한 연산자_를 오버로드할 수
있습니다. 다음 C# 예제를 살펴보겠습니다.

```csharp
Console.WriteLine(new Fraction(5, 4) + new Fraction(1, 2));  // 14/8

public readonly record struct Fraction(int Numerator, int Denominator)
{
    public static Fraction operator +(Fraction a, Fraction b) =>
        new(a.Numerator * b.Denominator + b.Numerator * a.Denominator, a.Denominator * b.Denominator);

    public override string ToString() => $"{Numerator}/{Denominator}";
}
```

Rust의 여러 연산자는 [트레이트를 통해 오버로드할 수 있습니다][ops.rs].
연산자가 메서드 호출을 간단하게 표현하는 구문이기 때문입니다. 예를
들어 `a + b`의 `+` 연산자는 `add` 메서드를 호출합니다. 다음 예제와
[연산자 오버로딩] 문서를 참고할 수 있습니다.

```rust
use std::{fmt::{Display, Formatter, Result}, ops::Add};

struct Fraction {
    numerator: i32,
    denominator: i32,
}

impl Display for Fraction {
    fn fmt(&self, f: &mut Formatter<'_>) -> Result {
        f.write_fmt(format_args!("{}/{}", self.numerator, self.denominator))
    }
}

impl Add<Fraction> for Fraction {
    type Output = Fraction;

    fn add(self, rhs: Fraction) -> Fraction {
        Fraction {
            numerator: self.numerator * rhs.denominator + rhs.numerator * self.denominator,
            denominator: self.denominator * rhs.denominator,
        }
    }
}

fn main() {
    println!(
        "{}",
        Fraction { numerator: 5, denominator: 4 } + Fraction { numerator: 1, denominator: 2 }
    ); // 14/8
}

```

[ops.rs]: https://doc.rust-lang.org/core/ops/
[연산자 오버로딩]: https://doc.rust-lang.org/rust-by-example/trait/ops.html
