# 소개

이 안내서는 Rust를 처음 접하는 C# 및 .NET 개발자를 대상으로 합니다. 모든
주제를 망라하지는 않습니다. C#/.NET과 Rust는 일부 개념과 구문이 비슷하지만
표현 방법은 다를 수 있습니다. 메모리 관리처럼 접근 방식 자체가 크게 다른
주제도 있습니다. 이 안내서는 두 언어의 주요 개념을 짧은 예제와 함께 비교하고
대응 관계를 설명합니다.

원저자[^authors]들도 Rust를 처음 접한 C#/.NET 개발자였습니다. 이 안내서는
원저자들이 몇 달간 Rust 코드를 작성하며 익힌 내용을 모았습니다. 원저자들은
Rust를 배우기 시작할 때 이와 같은 자료가 있었다면 도움이 되었을 것이라고
말합니다. 다만 C#과 .NET의 관점에만 기대어 Rust를 익히기보다는 Rust의 책과
웹 자료를 함께 읽으며 Rust 특유의 표현 방식도 접할 수 있습니다. 이 안내서는
상속, 스레드, 비동기 프로그래밍 등을 Rust에서 지원하는지 빠르게 알아보는 데
도움을 줍니다.

## 독자에 관한 전제

- C# 및 .NET 개발 경험이 풍부한 독자
- Rust를 처음 접하는 독자

## 이 안내서의 목표

- C#/.NET의 여러 주제와 Rust의 대응 개념을 간결하게 비교
- 추가 학습에 활용할 Rust 참조 문서, 책, 기술 문서 링크 제공

## 다루지 않는 내용

- 디자인 패턴과 아키텍처에 관한 설명
- Rust 언어를 처음부터 가르치는 튜토리얼
- 이 안내서만으로 Rust에 능숙해지는 학습 과정
- C#과 Rust의 코드 작성법을 폭넓게 모은 요리책 형식의 자료

---

[^authors]: 원저자는 알파벳순으로 [Atif Aziz], [Bastian Burger],
    [Daniele Antonio Maggio], [Dariusz Parys], [Patrick Schuler]입니다.

  [Atif Aziz]: https://github.com/atifaziz
  [Bastian Burger]: https://github.com/bastbu
  [Daniele Antonio Maggio]: https://github.com/danigian
  [Dariusz Parys]: https://github.com/dariuszparys
  [Patrick Schuler]: https://github.com/p-schuler
