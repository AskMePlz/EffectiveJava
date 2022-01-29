# Item9. try-finally 보다는 try-with-resources 를 사용하라

자바 라이브러리에서는 close 메소드를 호출해서 직접 닫아줘야 자원이 많다. 예를 들어, InputStream, OutputStream, java.sql.Connection 이 있다.
그런데 자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어질 수 있다. 이런 자원 중 상당수가 안정망으로 finalizer를 활용하고는 있지만 그리 믿을만 하지 못한다.

### 자원을 닫는 전통적인 방법 try-finally
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데, 예컨데,기기에 물리적인 문제가 생긴다면 firstLineOfFile 메서드 내의
readLine 메서드가 예외를 던지고, 같은 이유로 close 메서드도 실패할 것이다. 이런 상황이라면 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다.
그러면 스택 추적 내역에 첫번 째, 예외에 대해 남아있지 않게 되어 디버깅을 어렵게 한다.

이러한 문제들은 자바 7이 투척한 try-with-resources 덕분에 해결되었다.
### try-with-resources
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
이 구조를 사용하려면 `AutoCloseable` 인터페이스를 구현해야 한다.

readLine과 코드에는 보이지 않는 close 모두에서 예외가 발생한다면, close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다.
이렇게 숨겨진 예외들은 그냥 버려지지 않고 숨겨졌다는 꼬리표인 suppressed를 달고 출력된다. 또한 getSuppressed 메서드를 이용해 프로그램 코드에서 가져올 수 있다.
