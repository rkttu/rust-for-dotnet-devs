# 컴파일과 빌드

## .NET CLI와 Cargo

Rust의 [Cargo][cargo](`cargo`)는 .NET CLI(`dotnet`)에 대응합니다. 두 도구 모두 하위 도구를 쉽게 사용할 수 있는 진입점을 제공합니다. C# 컴파일러(`csc`)나 MSBuild(`dotnet msbuild`)를 직접 실행할 수도 있지만 보통 `dotnet build`로 솔루션을 빌드합니다. Rust에서도 컴파일러(`rustc`)를 직접 호출할 수 있지만 일반적으로 `cargo build`가 간편합니다.

[cargo]: https://doc.rust-lang.org/cargo/

## 빌드 결과물

.NET에서 [`dotnet build`][net-build-output]로 실행 프로그램을 빌드하면 패키지를 복원하고 소스 코드를 컴파일하여 [어셈블리][assembly]를 만듭니다. 어셈블리에는 중간 언어(IL) 코드가 들어 있습니다. 호스트에 .NET 런타임이 설치되어 있다면 일반적으로 .NET이 지원하는 여러 플랫폼에서 실행할 수 있습니다. 의존 패키지의 어셈블리는 대개 프로젝트 결과물과 함께 배치합니다. Rust의 [`cargo build`][cargo-build]도 빌드 결과물을 만들지만 Rust 컴파일러는 일반적으로 모든 코드를 정적으로 링크하여 플랫폼별 바이너리 하나로 만듭니다. 다른 [링크 방식][linkage]도 사용할 수 있습니다.

.NET에서는 `dotnet publish`로 프레임워크 종속 배포(FDD)나 자체 포함 배포(SCD)에 필요한 실행 파일을 준비합니다. Rust에는 이에 직접 대응하는 `dotnet publish` 명령이 없습니다. 빌드 결과물이 대상 플랫폼용 바이너리를 이미 포함하기 때문입니다.

.NET 라이브러리를 [`dotnet build`][net-build-output]로 빌드해도 IL을 포함한 [어셈블리][assembly]가 생성됩니다. Rust 라이브러리를 빌드하면 각 라이브러리 타깃에 해당하는 플랫폼별 컴파일 결과물이 생성됩니다.

관련 개념은 [크레이트][crate] 문서에서 확인할 수 있습니다.

[net-build-output]: https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-build#description
[assembly]: https://learn.microsoft.com/en-us/dotnet/standard/assembly/
[cargo-build]: https://doc.rust-lang.org/cargo/commands/cargo-build.html#cargo-build1
[linkage]: https://doc.rust-lang.org/reference/linkage.html
[crate]: https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html

## 의존성

.NET에서는 프로젝트 파일에 빌드 옵션과 의존성을 정의합니다. Cargo를 사용하는 Rust 프로젝트는 `Cargo.toml`에 패키지 의존성을 선언합니다. 다음은 일반적인 .NET 프로젝트 파일의 예입니다.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="morelinq" Version="3.3.2" />
  </ItemGroup>

</Project>
```

이에 대응하는 Rust의 `Cargo.toml`은 다음과 같이 작성합니다.

```toml
[package]
name = "hello_world"
version = "0.1.0"

[dependencies]
tokio = "1.0.0"
```

Cargo는 `src/main.rs`를 패키지와 이름이 같은 바이너리 크레이트의 루트로 취급합니다. 패키지에 `src/lib.rs`가 있으면 패키지와 이름이 같은 라이브러리 크레이트도 포함된 것으로 취급합니다.

## 패키지

.NET에서는 주로 NuGet으로 패키지를 설치합니다. 예를 들어 .NET CLI에서 NuGet 패키지 참조를 추가하면 프로젝트 파일에 의존성이 기록됩니다.

    dotnet add package morelinq

Rust에서도 Cargo로 비슷하게 패키지를 추가할 수 있습니다.

    cargo add tokio

.NET 패키지는 주로 [nuget.org]에, Rust 패키지는 주로 [crates.io]에 게시합니다.

[nuget.org]: https://www.nuget.org/
[crates.io]: https://crates.io

## 정적 코드 분석

.NET 5부터 .NET SDK는 코드 품질과 코드 스타일을 분석하는 Roslyn 분석기를 포함합니다. Rust에서 이에 대응하는 린트 도구는 [Clippy]입니다.

.NET 프로젝트에서 [`TreatWarningsAsErrors`][treat-warnings-as-errors]를 `true`로 설정해 경고가 있을 때 빌드를 실패시킬 수 있습니다. Clippy도 `cargo clippy -- -D warnings`로 컴파일러나 Clippy의 경고를 오류로 처리할 수 있습니다.

Rust CI 파이프라인에는 다음 정적 검사를 추가할 수 있습니다.

- [`cargo doc`][cargo-doc]으로 문서 빌드 검사
- [`cargo check --locked`][cargo-check]로 `Cargo.lock`의 최신 상태 검사

[clippy]: https://github.com/rust-lang/rust-clippy
[treat-warnings-as-errors]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/errors-warnings
[cargo-doc]: https://doc.rust-lang.org/cargo/commands/cargo-doc.html
[cargo-check]: https://doc.rust-lang.org/cargo/commands/cargo-check.html#manifest-options
