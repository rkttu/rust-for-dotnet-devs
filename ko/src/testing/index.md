# 테스트

## 테스트 구성

.NET 솔루션은 xUnit, NUnit, MSTest 등 어떤 프레임워크를 쓰는지와
단위 테스트인지 통합 테스트인지에 관계없이 보통 테스트 코드를 별도
프로젝트에 둡니다. 따라서 테스트할 애플리케이션이나 라이브러리와
테스트 코드는 서로 다른 어셈블리에 들어갑니다.

Rust에서는 _단위 테스트_를 관례적으로 `tests`라는 하위 모듈에
작성합니다. 이 모듈은 테스트 대상 애플리케이션 또는 라이브러리
모듈과 같은 _소스 파일_에 놓입니다. 이 구성에는 두 가지 장점이
있습니다.

- 구현 코드와 단위 테스트를 나란히 배치
- 하위 모듈이 내부 항목에 접근할 수 있으므로 .NET의
  `[InternalsVisibleTo]` 같은 우회 수단이 불필요

테스트 하위 모듈에는 `#[cfg(test)]` 특성을 붙입니다. 그러면
`cargo test`를 실행할 때만 전체 모듈을 조건부로 컴파일하고
실행합니다. 하위 모듈의 테스트 함수에는 `#[test]` 특성을
붙입니다.

통합 테스트는 보통 소스가 있는 `src` 디렉터리와 나란한 `tests`
디렉터리에 둡니다. `cargo test`는 그 디렉터리의 파일을 각각 별도
crate로 컴파일하고 `#[test]`가 붙은 메서드를 실행합니다.
`tests` 디렉터리의 파일은 통합 테스트로 인식하므로 모듈에
`#[cfg(test)]`를 붙이지 않아도 됩니다.

관련 자료:

- [테스트 구성][test-org]

[test-org]: https://doc.rust-lang.org/book/ch11-03-test-organization.html

## 테스트 실행

Rust에서 `dotnet test`에 대응하는 명령은 `cargo test`입니다.

`cargo test`는 기본적으로 모든 테스트를 병렬로 실행합니다.
스레드 하나로 순차 실행하려면 다음 명령을 사용합니다.

    cargo test -- --test-threads=1

자세한 내용은 [테스트 병렬 또는 순차 실행][tests-exec] 절에서
확인할 수 있습니다.

[tests-exec]: https://doc.rust-lang.org/book/ch11-02-running-tests.html#running-tests-in-parallel-or-consecutively

## 테스트 출력

복잡한 통합 테스트나 종단 간 테스트에서는 실행 중 일어난 일을
기록하기도 합니다. .NET에서는 프레임워크마다 방법이 다릅니다.
예를 들어 NUnit은 `Console.WriteLine`을 사용하고 xUnit은
`ITestOutputHelper`를 사용합니다. Rust에서는 NUnit과 비슷하게
`println!`으로 표준 출력에 기록합니다. 테스트 중 캡처한 출력은
기본적으로 보이지 않습니다. 테스트 프로그램에 `--show-output`을
전달하면 성공한 테스트의 출력도 볼 수 있습니다.

    cargo test -- --show-output

자세한 내용은 [함수 출력 표시][test-output] 절에서 확인할 수
있습니다.

[test-output]: https://doc.rust-lang.org/book/ch11-02-running-tests.html#showing-function-output

## 단언

.NET에서는 테스트 프레임워크에 따라 다양한 단언 방법을 사용합니다.
다음은 xUnit.net의 단언 예제입니다.

```csharp
[Fact]
public void Something_Is_The_Right_Length()
{
    var value = "something";
    Assert.Equal(9, value.Length);
}
```

Rust에서는 별도의 테스트 프레임워크나 crate가 없어도 됩니다.
표준 라이브러리가 대부분의 테스트에 쓸 수 있는 단언 _매크로_를
제공합니다.

- [`assert!`][assert]
- [`assert_eq!`][assert_eq]
- [`assert_ne!`][assert_ne]

다음 예제에서는 `assert_eq`를 사용합니다.

```rust
#[test]
fn something_is_the_right_length() {
    let value = "something";
    assert_eq!(9, value.len());
}
```

Rust 표준 라이브러리에는 xUnit.net의 `[Theory]` 같은 데이터
기반 테스트 기능이 없습니다.

[assert]: https://doc.rust-lang.org/std/macro.assert.html
[assert_eq]: https://doc.rust-lang.org/std/macro.assert_eq.html
[assert_ne]: https://doc.rust-lang.org/std/macro.assert_ne.html

## 모의 객체

.NET 애플리케이션이나 라이브러리를 테스트할 때 Moq, NSubstitute
같은 프레임워크로 타입의 종속성을 대체할 수 있습니다. Rust에도
[`mockall`][mockall] 같은 crate가 있습니다. 외부 crate 없이
[`cfg` 특성][cfg-attribute]에 기반한 [조건부 컴파일]로 간단히
대체할 수도 있습니다. `cfg`는 `test` 같은 구성 기호에 따라
표시된 코드를 조건부로 포함합니다. `DEBUG`에 따라 디버그
빌드용 코드를 컴파일하는 것과 비슷합니다. 이 방식은 모듈의 모든
테스트가 하나의 구현만 사용할 수 있다는 제약이 있습니다.

`#[cfg(test)]`를 붙인 코드는 `cargo test`를 실행할 때만
컴파일하고 실행합니다. 내부적으로는 `rustc --test`로 컴파일합니다.
반대로 `#[cfg(not(test))]`가 붙은 코드는 `cargo test`로
테스트하지 않을 때만 포함합니다.

다음 예제는 환경 변수 값을 읽어 반환하는 표준 라이브러리의
`var_os` 함수를 대체합니다. `get_env`가 사용하는 `var_os`의
모의 버전을 조건부로 가져옵니다. `cargo build`나 `cargo run`
에서는 `std::env::var_os`를 사용합니다. `cargo test`에서는
`tests::var_os_mock`을 `var_os`라는 이름으로 가져와 테스트
중에 모의 구현을 사용합니다.

```rust
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT license.

/// Utility function to read an environment variable and return its value if
/// defined. It fails/panics if the value is not valid Unicode.
pub fn get_env(key: &str) -> Option<String> {
    #[cfg(not(test))]                 // for regular builds...
    use std::env::var_os;             // ...import from the standard library
    #[cfg(test)]                      // for test builds...
    use tests::var_os_mock as var_os; // ...import mock from test sub-module

    let val = var_os(key);
    val.map(|s| s.to_str()     // get string slice
                 .unwrap()     // panic if not valid Unicode
                 .to_owned())  // convert to "String"
}

#[cfg(test)]
mod tests {
    use std::ffi::*;
    use super::*;

    pub(crate) fn var_os_mock(key: &str) -> Option<OsString> {
        match key {
            "FOO" => Some("BAR".into()),
            _ => None
        }
    }

    #[test]
    fn get_env_when_var_undefined_returns_none() {
        assert_eq!(None, get_env("???"));
    }

    #[test]
    fn get_env_when_var_defined_returns_some_value() {
        assert_eq!(Some("BAR".to_owned()), get_env("FOO"));
    }
}
```

[mockall]: https://docs.rs/mockall/latest/mockall/
[조건부 컴파일]: ../conditional-compilation/index.md
[cfg-attribute]: https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-attribute

## 코드 적용 범위

.NET에는 테스트 코드 적용 범위를 분석하는 도구가 다양합니다.
Visual Studio는 도구를 내장하고 Visual Studio Code에는 확장이
있습니다. .NET 개발자는 [coverlet]에도 익숙할 수 있습니다.

Rust는 테스트 코드 적용 범위를 수집할 [내장 기능][built-in-cov]을
제공합니다. 추가 분석 도구도 있지만 통합 방식이 완전히 자동화되어
있지는 않습니다. 몇 가지 수동 단계를 거치면 결과를 시각적으로
확인할 수 있습니다.

Visual Studio Code의 [Coverage Gutters][coverage.gutters] 확장과
[Tarpaulin]을 함께 사용하면 코드 적용 범위를 편집기에서 볼 수
있습니다. Coverage Gutters에는 LCOV 파일이 필요합니다. 이
파일은 Tarpaulin 이외의 도구로도 만들 수 있습니다.

설정을 마친 뒤 다음 명령을 실행합니다.

```bash
cargo tarpaulin --ignore-tests --out Lcov
```

이 명령은 LCOV 코드 적용 범위 파일을 만듭니다. `Coverage Gutters:
Watch`를 켜면 확장이 파일을 읽고 소스 편집기에 줄별 적용 범위를
표시합니다.

> LCOV 파일의 위치가 중요합니다. 여러 패키지가 있는 [프로젝트
> 구조]에서 `--workspace`로 루트에 LCOV 파일을 만들면 패키지
> 루트에 별도 파일이 있더라도 워크스페이스 루트의 파일을 사용합니다.
> 루트 전체의 파일을 만드는 것보다 테스트 대상 패키지만
> 분리해 실행하는 편이 더 빠릅니다.

[coverage.gutters]: https://marketplace.visualstudio.com/items?itemName=ryanluker.vscode-coverage-gutters
[tarpaulin]: https://github.com/xd009642/tarpaulin
[coverlet]: https://github.com/coverlet-coverage/coverlet
[built-in-cov]: https://doc.rust-lang.org/stable/rustc/instrument-coverage.html#test-coverage
[프로젝트 구조]: ../project-structure/index.md
