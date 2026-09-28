# 환경 변수와 구성

## 환경 변수에 접근하기

.NET에서는 `System.Environment.GetEnvironmentVariable` 메서드로 실행 중인 프로세스의 환경 변수 값을 읽습니다.

```csharp
using System;

const string name = "EXAMPLE_VARIABLE";

var value = Environment.GetEnvironmentVariable(name);
if (string.IsNullOrEmpty(value))
    Console.WriteLine($"Variable '{name}' not set.");
else
    Console.WriteLine($"Variable '{name}' set to '{value}'.");
```

Rust에서는 `std::env` 모듈의 `var`와 `var_os` 함수로 실행 중에 환경 변수에 접근합니다.

`var`는 `Result<String, VarError>`를 반환합니다. 환경 변수가 설정되어 있으면 값을 반환하고, 설정되지 않았거나 유효한 유니코드가 아니면 오류를 반환합니다.

`var_os`는 `Option<OsString>`을 반환합니다. 변수가 설정되어 있으면 `Some` 값을, 설정되지 않았으면 `None`을 반환합니다. `OsString`은 유효한 유니코드일 필요가 없습니다.

```rust
use std::env;


fn main() {
    let key = "ExampleVariable";
    match env::var(key) {
        Ok(val) => println!("{key}: {val:?}"),
        Err(e) => println!("couldn't interpret {key}: {e}"),
    }
}
```

```rust
use std::env;

fn main() {
    let key = "ExampleVariable";
    match env::var_os(key) {
        Some(val) => println!("{key}: {val:?}"),
        None => println!("{key} not defined in the environment"),
    }
}
```

Rust에서는 컴파일 시점에도 환경 변수에 접근할 수 있습니다. `env!` 매크로는 컴파일할 때 환경 변수 값을 `&'static str`로 확장합니다. 해당 변수가 설정되지 않았다면 컴파일 오류가 발생합니다.

```rust
use std::env;

fn main() {
    let example = env!("ExampleVariable");
    println!("{example}");
}
```

.NET에서도 [소스 생성기][source-gen]를 활용하면 컴파일 시점에 환경 변수에 접근할 수 있습니다. 다만 Rust의 `env!`처럼 직접 사용하는 방식은 아닙니다.

[source-gen]: https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview

## 구성

.NET에서는 구성 공급자로 설정 값을 읽습니다. 프레임워크는 `Microsoft.Extensions.Configuration` 네임스페이스와 NuGet 패키지를 통해 여러 공급자 구현을 제공합니다.

구성 공급자는 다양한 원천의 키와 값 쌍을 읽고 `IConfiguration`을 통해 통합된 설정을 제공합니다.

```csharp
using Microsoft.Extensions.Configuration;

class Example {
    static void Main()
    {
        IConfiguration configuration = new ConfigurationBuilder()
            .AddEnvironmentVariables()
            .Build();

        var example = configuration.GetValue<string>("ExampleVar");

        Console.WriteLine(example);
    }
}
```

다른 공급자는 공식 [.NET 구성 공급자 문서][conf-net]에서 살펴볼 수 있습니다.

Rust에서는 [figment]나 [config] 같은 외부 크레이트를 사용해 비슷한 방식으로 구성을 관리할 수 있습니다. 다음 예제는 [config] 크레이트를 사용합니다.

```rust
use config::{Config, Environment};

fn main() {
    let builder = Config::builder().add_source(Environment::default());

    match builder.build() {
        Ok(config) => {
            match config.get_string("examplevar") {
                Ok(v) => println!("{v}"),
                Err(e) => println!("{e}")
            }
        },
        Err(_) => {
            // 오류가 발생했습니다.
        }
    }
}

```

[conf-net]: https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration-providers
[figment]: https://crates.io/crates/figment
[config]: https://crates.io/crates/config
