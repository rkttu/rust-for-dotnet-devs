# 벤치마킹

Rust에서는 [`cargo bench`][cargo-bench]로 벤치마크를 실행합니다.
이 Cargo 명령은 `#[bench]` 특성이 붙은 메서드를 실행합니다.
원문 작성 당시 이 특성은 [불안정 기능][bench-unstable]이며
nightly 채널에서만 사용할 수 있었습니다.

.NET에서는 `BenchmarkDotNet` 라이브러리로 메서드 성능을 측정하고
추적할 수 있습니다. Rust에서 비슷한 역할을 하는 crate는
`Criterion`입니다.

[Criterion 문서][criterion-docs]에 따르면 실행 간 통계 정보를
수집하고 저장합니다. 성능 저하를 자동으로 감지하고 최적화
결과를 측정할 수 있습니다.

`Criterion`은 `#[bench]` 특성 대신 `criterion_group!`과
`criterion_main!` 매크로를 사용합니다. [Criterion 시작하기][criterion-start]
문서에 나온 설정으로 안정 채널에서도 벤치마크를 실행할 수 있습니다.

`BenchmarkDotNet`처럼 벤치마크 결과를 [지속적 벤치마킹용 GitHub
Action][gh-action-bench]과 통합할 수도 있습니다. `Criterion`은
여러 출력 형식을 지원합니다. 그중 `bencher` 형식은 nightly의
`libtest` 벤치마크 형식을 모방하며 해당 Action과 호환됩니다.

[cargo-bench]: https://doc.rust-lang.org/cargo/commands/cargo-bench.html
[bench-unstable]: https://doc.rust-lang.org/rustc/tests/index.html#test-attributes
[criterion-docs]: https://bheisler.github.io/criterion.rs/book/index.html
[criterion-start]: https://bheisler.github.io/criterion.rs/book/getting_started.html
[gh-action-bench]: https://github.com/benchmark-action/github-action-benchmark
