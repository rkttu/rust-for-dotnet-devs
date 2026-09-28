# 리소스 관리

앞의 [메모리 관리] 장에서는 GC, 소유권, 종료자를 중심으로 .NET과
Rust의 차이를 설명했습니다. 여기서는 리소스 해제 예제를 살펴보겠습니다.

다음 예제는 가상의 SQL _데이터베이스 연결_을 만들고 올바르게
닫거나 해제하거나 드롭하는 방법을 보여 줍니다. 먼저 .NET
코드입니다.

```csharp
{
    using var db1 = new DatabaseConnection("Server=A;Database=DB1");
    using var db2 = new DatabaseConnection("Server=A;Database=DB2");

    // ..."db1"과 "db2"를 사용하는 코드...
}   // "db1"과 "db2"가 범위를 벗어나 여기서 Dispose를 호출합니다.

public class DatabaseConnection : IDisposable
{
    readonly string connectionString;
    SqlConnection connection; // IDisposable을 구현합니다.

    public DatabaseConnection(string connectionString) =>
        this.connectionString = connectionString;

    public void Dispose()
    {
        // SqlConnection을 반드시 해제합니다.
        this.connection.Dispose();
        Console.WriteLine("Closing connection: {this.connectionString}");
    }
}
```

Rust에서는 같은 리소스를 다음과 같이 다룹니다.

```rust
struct DatabaseConnection(&'static str);

impl DatabaseConnection {
    // ...데이터베이스 연결을 사용하는 함수...
}

impl Drop for DatabaseConnection {
    fn drop(&mut self) {
        // ...연결을 닫는 코드...
        self.close_connection();
        // ...메시지를 출력하는 코드...
        println!("Closing connection: {}", self.0)
    }
}

fn main() {
    let _db1 = DatabaseConnection("Server=A;Database=DB1");
    let _db2 = DatabaseConnection("Server=A;Database=DB2");
    // ...데이터베이스 연결을 사용하는 코드...
} // "db1"과 "db2"가 범위를 벗어나 여기서 Dispose를 호출합니다.
```

.NET에서 `Dispose`를 호출한 뒤 객체를 다시 사용하면 보통 실행
중에 `ObjectDisposedException`이 발생합니다. Rust는 같은 종류의
잘못된 사용을 컴파일할 때 차단합니다.

[메모리 관리]: ../memory-management/index.md
