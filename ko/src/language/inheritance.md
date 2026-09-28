# 상속

[구조체] 절에서 설명하듯 Rust에는 C#과 같은 클래스 상속이 없습니다.
구조체 간에 동작을 공유하려면 트레이트를 사용할 수 있습니다. 또한 C#의
_인터페이스 상속_과 비슷하게 Rust에서도 [_슈퍼트레이트_][supertrait.rs]로
트레이트 사이의 관계를 정의할 수 있습니다.

[구조체]: ./custom-types/structs.md
[supertrait.rs]: https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#using-supertraits-to-require-one-traits-functionality-within-another-trait
