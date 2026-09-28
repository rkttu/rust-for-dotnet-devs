# 네임스페이스

.NET은 네임스페이스로 타입을 구성하고 프로젝트에서 타입과 메서드가 보이는
범위를 제어합니다.

Rust에서 네임스페이스라는 말은 다른 개념을 가리킵니다. .NET 네임스페이스에
대응하는 Rust의 기능은 [모듈][rust-module]입니다. C#과 Rust는 각각 접근
한정자와 가시성 한정자로 항목의 공개 범위를 제한합니다. Rust에서는 몇 가지
예외를 제외하면 기본 가시성이 _비공개_입니다. C#의 `public`은 Rust의
`pub`에, `internal`은 `pub(crate)`에 대응합니다. 세부 접근 범위는
[가시성 한정자] 문서에서 확인할 수 있습니다.

[rust-module]: https://doc.rust-lang.org/reference/items/modules.html
[가시성 한정자]: https://doc.rust-lang.org/reference/visibility-and-privacy.html
