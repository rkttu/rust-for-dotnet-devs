# 람다와 클로저

C#과 Rust는 함수를 일급 값으로 다루며 _고차 함수_를 작성할 수 있습니다.
고차 함수는 다른 함수를 인수로 받아 호출자가 함수 동작의 일부를 제공하도록
합니다. C#에서는 타입 안전한 함수 참조를 대리자로 표현하며, `Func`와
`Action`을 자주 사용합니다. _람다 식_으로 대리자 인스턴스를 즉석에서
만들 수 있습니다.

Rust에도 함수 포인터가 있습니다. 가장 간단한 형태는 `fn` 타입입니다.

```rust
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(|x| x + 1, 5);
    println!("The answer is: {}", answer); // 출력: The answer is: 12
}
```

Rust는 _함수 포인터_와 _클로저_를 구분합니다. 함수 포인터의 타입은
`fn`으로 정의합니다. 클로저는 주변 어휘 범위의 변수를 참조할 수 있지만
함수 포인터는 그럴 수 없습니다. C#에도 `delegate*` 형태의
[함수 포인터][*delegate]가 있습니다. 관리 코드에서 타입 안전하게
사용하는 정적 람다 식은 Rust의 함수 포인터와 더 가깝습니다.

[*delegate]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code#function-pointers

클로저를 받는 함수와 메서드는 `Fn`, `FnMut`, `FnOnce` 트레이트 중 하나를
제약 조건으로 둔 제네릭 타입을 사용합니다. 함수 포인터나 클로저 값을
전달할 때는 앞 예제의 `|x| x + 1` 같은 _클로저 식_을 작성합니다.
C#의 람다 식과 역할이 같습니다. 클로저 식이 함수 포인터가 될지 클로저가
될지는 주변 변수를 참조하는지에 따라 달라집니다.

클로저가 주변 변수를 캡처하면 소유권 규칙도 적용됩니다. 캡처한 값의
소유권이 클로저로 이동할 수 있기 때문입니다. 자세한 내용은 The Rust
Programming Language의 [클로저에서 캡처한 값 이동과 `Fn` 트레이트][closure-move]
절에서 확인할 수 있습니다.

[closure-move]: https://doc.rust-lang.org/book/ch13-01-closures.html#moving-captured-values-out-of-closures-and-the-fn-traits
