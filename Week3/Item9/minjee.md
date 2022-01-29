#### 아이템9
# try-finally 보다는 try-with-resources를 사용하라

###  close 메서드 호출, 직접 닫아줘야하는 경우
- InputStream, OuputStream, java.sql.Connection 등
- 자원닫기를 위해 안전망으로 finalizer를 활용하고 있으나 그리 믿을만하지 못함

### try-finally
- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 사용
- 예외 발생, 메서드 반환 포함

```
   // 예제 9-1
   static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
    
    //예제 9-2
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                out.close();
            }
        } finally {
            in.close();
        }
    }
```
- try 블록과 finally 블록에서 모두 예외가 발생할 수 있는데 두번째 발생한 예외가 첫번쨰 발생한 예외를 집어삼켜 첫번째 예외에 대한 정보가 안남는다
   - 예제 9-1 에서 reaLine 에서 예외를 던지고 같은 이유로 close 메서드도 실패, 그러나 두번째 예외만 남아 디버깅에 어려움을 겪을 수 있음
   - 두번째 예외 대신 첫번째 예외를 기록하도록 코드를 수정할 수 있으나 코드가 너무 지저분해져서 실제로 그렇게 하는 경우는 거의 없음

### try-with-resources
- 앞의 문제를 해결
- *try(...) 안에 객체 선언 및 할당하면 try안에서 사용이 가능하며 코드의 실행위치가 try문을 벗어나면 try(...)안에 선언된 객체의 close() 메서드를 호출*
   - *finally에서 명시적으로 close()를 호출할 필요가 없음*
- AutoCloseable 인터페이스를 구현한 객체만이 close를 자동 호출
   - void를 반환하는 close 메서드만 정의한 인터페이스
   - 수많은 클래스, 인터페이스가 이미 AutoCloseable 을 구현 또는 확장
   - 닫아야하는 자원을 뜻하는 클래스를 작성한다면 반드시 AutoCloseable 구현 필요
- 짧고 읽기 쉬우며 문제를 진단하기도 수월함

- 앞의 예제 수정한 코드

```
//예제 9-4
    static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        }
    }

//예제 9-4
    static void copy(String src, String dst) throws IOException {
        try (InputStream   in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
```
- 예제 9-3에서 readLine(코드에는 나타나지 않는)과 close 호출 양쪽에서 예외가 발생할 경우
   - close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록
   - 숨겨진 예외들은 그냥 버려지지 않고 스택 추적 내역에 "숨겨졌다 - suppresse"라는 꼬리표를 달고 출력
   - 자바 7에서 Throwable에 추가된 getSuppressed 메서드를 이용하면 프로그래 코드를 가져올 수 있음

###  + catch
- try-with-resources에서도 catch 절 사용 가능
- catch 절덕분에 try 문을 중첩하지 않고 다수 예외 처리 가능

```
    static String firstLineOfFile(String path, String defaultVal) {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return defaultVal;
        }
    }
```
- 예제 9-3/9-1을 수정한 예시
- 파일을 열거나 데이터를 읽지 못할 때 예외를 던지는 대신 기본값을 반환하도록 

>> 꼭 회수해야 하는 자원을 다룰 때에는 try-with-resources를 사용하자 
>> 예외는 없고 코드는 더 짧고 분명해지며 만들어지는 예외정보도 훨씬 유용하다
>> try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도
>> try-with-resources 로는 정확하고 쉽게 자원을 회수할 

##### 참고
try-with-resources 추가 설명 https://codechacha.com/ko/java-try-with-resources/
