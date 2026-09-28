# 한국어판 번역 기록

2026년 9월 28일, 한국어판은 원본 저장소의 [`0991644` 커밋](https://github.com/microsoft/rust-for-dotnet-devs/commit/0991644c384045d42dac72564e92d3fee6d3a9b7)을 기준으로 책 전체를 번역했습니다. 이 기록은 원본과의 대응 관계, 검토 중 수정한 내용, 검증 결과, 단계별 커밋을 담습니다.

한국어판은 원문의 Markdown 파일 46개에 모두 대응합니다. 설명 문장과 예제의 설명 주석을 번역했고, 원문의 코드 동작과 법적 고지를 보존했습니다. 원본에서 확인한 오류 네 가지를 바로잡았습니다. 아래에서는 번역 범위와 보존 원칙을 먼저 설명한 뒤 수정 사항, 검증 방법, 커밋 순서를 정리하겠습니다.

## 원문과 대응하는 46개 Markdown 파일

한국어판의 [`src/`](src/SUMMARY.md)는 원본 [`src/`](https://github.com/microsoft/rust-for-dotnet-devs/tree/0991644c384045d42dac72564e92d3fee6d3a9b7/src)와 같은 상대 경로에 Markdown 파일 46개를 둡니다. [`SUMMARY.md`](src/SUMMARY.md)는 모든 장의 제목을 한국어로 안내합니다. 루트의 영어 원문 `src/`는 변경하지 않았습니다.

## 코드와 라이선스의 보존

예제의 설명 주석은 한국어로 옮겼습니다. 코드의 식별자, 문자열 리터럴, 실행 결과는 예제 동작과 출력 비교를 위해 유지했습니다. 원문과 한국어판의 fenced 코드 블록을 비교할 때 주석을 제외한 코드 줄과 블록 개수가 모두 일치했습니다. 삽화 SVG도 원본을 사용합니다.

[`license.md`](src/license.md)의 제목은 번역했으며 MIT 허가 고지 본문과 저작권 표시는 원문 그대로 유지했습니다. 따라서 번역본을 배포할 때에도 원문의 허가 고지를 확인할 수 있습니다.

## 원문에서 바로잡은 네 가지 항목

1. [테스트 장](src/testing/index.md)은 테스트 실행 파일에 `--show-output`을 전달하는 명령을 `cargo test -- --show-output`으로 바로잡았습니다. 원문의 `cargo test --show-output`은 Cargo 옵션과 테스트 실행 파일 옵션의 경계를 빠뜨렸습니다. [The Rust Programming Language의 테스트 실행 설명](https://doc.rust-lang.org/book/ch11-02-running-tests.html)을 따랐습니다.
2. [벤치마킹 장](src/benchmarking/index.md)은 Criterion이 안정 Rust에서 `#[bench]`를 제공한다는 원문 설명을 고쳤습니다. Criterion 예제의 `criterion_group!`과 `criterion_main!`을 설명합니다. [Criterion 시작 문서](https://bheisler.github.io/criterion.rs/book/getting_started.html)를 따랐습니다.
3. [LINQ 장](src/linq/index.md)은 원문 표의 `Single`과 `SingleOrDefault`를 각각 Rust의 `find`와 `try_find`에 직접 대응시키던 항목을 고쳤습니다. [.NET `Single`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.single)과 [`SingleOrDefault`](https://learn.microsoft.com/en-us/dotnet/api/system.linq.enumerable.singleordefault)는 결과의 유일성을 검사하지만 [Rust `find`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.find)는 첫 번째 일치 항목을 반환합니다.
4. [LINQ 장](src/linq/index.md)의 `Vec<int>` 표기를 Rust의 타입 표기인 `Vec<i32>`로 바로잡았습니다. 원문의 예제는 정수 벡터를 사용합니다. Rust의 [`Vec` 문서](https://doc.rust-lang.org/std/vec/struct.Vec.html)를 기준으로 표기했습니다.

버전에 따라 달라질 수 있는 일부 설명은 원문 작성 당시의 상태임을 문장에 표시했습니다. 예를 들어 [반복자 메서드](src/linq/index.md#반복자-메서드yield)의 코루틴과 [비동기 반복](src/asynchronous-programming/index.md#비동기-반복)을 다루는 문장이 이에 해당합니다.

## 재현할 수 있는 검증

로컬에서는 원본 CI가 사용하는 mdBook 0.4.23으로 다음 명령을 실행했습니다.

```sh
mdbook build ko
```

빌드가 완료되었습니다. 파일 목록을 대조해 누락된 Markdown 파일이 없음을 확인했고, 한국어판의 내부 상대 링크를 검사해 누락된 대상이 없음을 확인했습니다. 원문과 한국어판의 fenced 코드 블록에서는 주석을 제외한 코드 줄이 모두 일치했습니다. 저장소의 `.markdownlint.jsonc` 설정으로 markdownlint-cli 0.49.1을 실행했으며, 한국어판과 README의 링크는 원본 CI가 사용하는 markdown-link-check 3.10.3으로 검사했습니다. 두 검사와 `git diff --check`를 모두 통과했습니다.

[GitHub Pages 미리보기](https://rkttu.github.io/rust-for-dotnet-devs/)는 포크의 [`ko-translation` 브랜치](https://github.com/rkttu/rust-for-dotnet-devs/tree/ko-translation)를 [Pages 워크플로](../.github/workflows/publish-ko-pages.yml)로 빌드해 게시합니다. `ko/` 또는 워크플로 파일을 이 브랜치에 푸시하면 다시 배포합니다. 첫 [배포 실행](https://github.com/rkttu/rust-for-dotnet-devs/actions/runs/36365073028)은 성공했고, 공개 사이트의 홈, 언어 장, CSS, 스크립트, 삽화는 HTTP 200으로 확인했습니다.

## 단계별 커밋 기록

각 단계는 [포크의 `ko-translation` 브랜치](https://github.com/rkttu/rust-for-dotnet-devs/tree/ko-translation)에 별도로 커밋하고 푸시했습니다.

| 커밋 | 작업 범위 |
| --- | --- |
| [`0289bd0`](https://github.com/rkttu/rust-for-dotnet-devs/commit/0289bd0) | 한국어 mdBook 시작, 소개, 시작하기, 라이선스 |
| [`0f0925c`](https://github.com/rkttu/rust-for-dotnet-devs/commit/0f0925c) | 전체 장 구조와 한국어 목차, 짧은 언어 장 |
| [`d101478`](https://github.com/rkttu/rust-for-dotnet-devs/commit/d101478) | 핵심 언어 개념 |
| [`835be05`](https://github.com/rkttu/rust-for-dotnet-devs/commit/835be05) | 문자열, 타입, 오류 처리 |
| [`76a3bd1`](https://github.com/rkttu/rust-for-dotnet-devs/commit/76a3bd1) | 메모리, 리소스, 스레딩 |
| [`5efe203`](https://github.com/rkttu/rust-for-dotnet-devs/commit/5efe203) | 테스트, 벤치마킹, 로깅 |
| [`dce6467`](https://github.com/rkttu/rust-for-dotnet-devs/commit/dce6467) | 프로젝트, 빌드, 설정, 메타프로그래밍 |
| [`408b584`](https://github.com/rkttu/rust-for-dotnet-devs/commit/408b584) | LINQ와 반복자 |
| [`90caf92`](https://github.com/rkttu/rust-for-dotnet-devs/commit/90caf92) | 비동기 프로그래밍 |
| [`17b93e4`](https://github.com/rkttu/rust-for-dotnet-devs/commit/17b93e4) | 예제의 설명 주석 |
| [`78c65fc`](https://github.com/rkttu/rust-for-dotnet-devs/commit/78c65fc) | 전체 번역 범위와 검증 기록 |
| [`efd89eb`](https://github.com/rkttu/rust-for-dotnet-devs/commit/efd89eb) | 한국어판 GitHub Pages 배포 워크플로 |

## 원본 저장소 기여 논의

[원본 이슈 #47](https://github.com/microsoft/rust-for-dotnet-devs/issues/47)에 한국어판을 원본에 병합하고 싶다는 [의사 표시](https://github.com/microsoft/rust-for-dotnet-devs/issues/47#issuecomment-5861138501)를 남겼습니다. 원본 관리자의 답변과 원하는 번역 구조는 아직 확인되지 않았습니다. 이 브랜치는 포크에서 먼저 전체 번역을 검토할 수 있도록 구성했습니다.

## 마무리

여기까지 정리하면 한국어판은 원본 책의 모든 Markdown 파일과 장 구성을 포함하며, 단계별 커밋으로 번역 과정을 추적할 수 있습니다. 원본 내용의 날짜 민감한 설명은 원문 작성 시점을 밝혔고, 사용자가 바로 실행하는 명령과 대응 표의 오류는 공식 문서를 근거로 수정했습니다.

원본 병합 방식은 관리자의 답변에 따라 조정할 수 있습니다. 그전에도 포크의 `ko-translation` 브랜치에서 한국어판을 빌드하거나 GitHub Pages에서 읽을 수 있습니다.
