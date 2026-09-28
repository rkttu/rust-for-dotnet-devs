# 로깅과 추적

.NET은 여러 로깅 API를 제공합니다. `ILogger`는 내장 로깅
공급자와 타사 공급자를 폭넓게 지원하므로 대부분의 경우 기본
선택으로 사용할 수 있습니다. 다음은 C#의 구조화된 로깅 예제입니다.

```csharp
using Microsoft.Extensions.Logging;

using var loggerFactory = LoggerFactory.Create(builder => builder.AddConsole());
var logger = loggerFactory.CreateLogger<Program>();
logger.LogInformation("Hello {Day}.", "Thursday"); // Hello Thursday.
```

Rust의 [log][log.rs] crate는 가벼운 로깅 파사드를 제공합니다.
`ILogger`보다 기능이 적고 원문 작성 당시 안정적인 구조화 로깅이나
로깅 범위를 제공하지 않았습니다.

.NET과 더 비슷한 기능이 필요하다면 Tokio의 [`tracing`][tracing.rs]을
사용할 수 있습니다. `tracing`은 Rust 애플리케이션을 계측해 구조화된
이벤트 기반 진단 정보를 수집하는 프레임워크입니다.
[`tracing_subscriber`][tracing-subscriber.rs]로 `tracing`
구독자를 구현하고 조합할 수 있습니다. 앞의 구조화 로깅을 이 두
crate로 작성하면 다음과 같습니다.

```rust
fn main() {
    // install global default ("console") collector.
    tracing_subscriber::fmt().init();
    tracing::info!("Hello {Day}.", Day = "Thursday"); // Hello Thursday.
}
```

[OpenTelemetry][opentelemetry.rs]는 규격에 따라 원격 측정 데이터를
계측, 생성, 수집, 내보내기 위한 도구와 API, SDK를 제공합니다.
원문 작성 당시 [OpenTelemetry Logging API][opentelemetry-logging]는
안정화되지 않았고 Rust 구현은 [로깅을 지원하지
않았지만][opentelemetry-status.rs] 추적 API는 지원했습니다.

[opentelemetry.rs]: https://crates.io/crates/opentelemetry
[tracing-subscriber.rs]: https://docs.rs/tracing-subscriber/latest/tracing_subscriber/
[opentelemetry-logging]: https://opentelemetry.io/docs/reference/specification/status/#logging
[opentelemetry-status.rs]: https://opentelemetry.io/docs/instrumentation/rust/#status-and-releases
[tracing.rs]: https://crates.io/crates/tracing
[log.rs]: https://crates.io/crates/log
