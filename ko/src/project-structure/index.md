# 프로젝트 구조

.NET 프로젝트에도 일반적인 디렉터리 구성이 있지만 Rust 프로젝트보다 형식이 자유로운 편입니다. Visual Studio 2022에서 클래스 라이브러리와 xUnit 테스트 프로젝트로 구성된 솔루션을 만들면 다음과 같은 구조가 됩니다.

    .
    |   SampleClassLibrary.sln
    +---SampleClassLibrary
    |       Class1.cs
    |       SampleClassLibrary.csproj
    +---SampleTestProject
            SampleTestProject.csproj
            UnitTest1.cs
            Usings.cs

각 프로젝트는 자체 `.csproj` 파일과 별도 디렉터리를 사용합니다. 저장소 루트에는 `.sln` 파일이 있습니다.

Cargo는 새 [패키지][rust-package]의 구성을 쉽게 파악할 수 있도록 다음 [패키지 레이아웃][package layout]을 따릅니다.

    .
    +-- Cargo.lock
    +-- Cargo.toml
    +-- src/
    |   +-- lib.rs
    |   +-- main.rs
    +-- benches/
    |   +-- some-bench.rs
    +-- examples/
    |   +-- some-example.rs
    +-- tests/
        +-- some-integration-test.rs

- `Cargo.toml`과 `Cargo.lock`은 패키지 루트에 둡니다.
- `src/lib.rs`는 기본 라이브러리 파일이며 `src/main.rs`는 기본 실행 파일입니다. 자세한 규칙은 [타깃 자동 검색][target auto-discovery]을 참고할 수 있습니다.
- 벤치마크는 `benches`에, 통합 테스트는 `tests`에 둡니다. [테스트][section-testing]와 [벤치마킹][section-benchmarking] 장에서 자세히 다룹니다.
- 예제는 `examples`에 둡니다.
- 단위 테스트용 크레이트를 별도로 만들지 않습니다. 단위 테스트는 대상 코드와 같은 파일에 둡니다. [테스트][section-testing] 장에서 예제를 확인할 수 있습니다.

[package layout]: https://doc.rust-lang.org/cargo/guide/project-layout.html
[rust-package]: https://doc.rust-lang.org/cargo/appendix/glossary.html#package
[target auto-discovery]: https://doc.rust-lang.org/cargo/reference/cargo-targets.html#target-auto-discovery
[section-testing]: ../testing/index.md
[section-benchmarking]: ../benchmarking/index.md

## 대규모 프로젝트 관리

Rust의 대규모 프로젝트에서는 Cargo [워크스페이스][cargo-workspaces]로 관련 패키지를 함께 관리할 수 있습니다. 기본 패키지가 없는 프로젝트에서는 [_가상 매니페스트_][cargo-virtual-manifest]를 사용하기도 합니다.

[cargo-workspaces]: https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html
[cargo-virtual-manifest]: https://doc.rust-lang.org/cargo/reference/workspaces.html#virtual-workspace

## 의존성 버전 관리

대규모 .NET 프로젝트에서는 [중앙 패키지 관리][Central Package Management]로 의존성 버전을 한곳에서 관리할 수 있습니다. Cargo는 [워크스페이스 상속][workspace inheritance]으로 의존성을 중앙에서 관리합니다.

[Central Package Management]: https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management
[workspace inheritance]: https://doc.rust-lang.org/cargo/reference/workspaces.html#the-package-table
