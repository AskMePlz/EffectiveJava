# [item9]try-finally보다는 try-with-resources를 사용하라
### try-finally
전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다.

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

위의 코드가 있다고하자, 기기의 물리적인 문제로 인해 try 블록 안에서 예외를 던지고, finally 블록 안에서도 close 메서드가 실패한다면 finally 블록에서 발생한 예외가 try블록 안에서의 예외를 먹어서 try블록 안의 예외에 대한 정보를 얻을 수 없는 상태가 된다. 
#
### try-with-resources

 위에서 다루었던 try-catch의 문제점은 try-with-resources 덕에 모두 해결할 수 있었다. 아래 코드는 위에 있는 코드를 try-with-resources로 바꾼 코드다. 코드가 더 짧고 분명해진 것을 확인할 수 있다.

 이것 뿐만이 아니다. 위에서 2개의 예외가 동시에 발생되었을때, try-finally 같은 경우에는 close 예외에 먹혀서 첫번째 예외가 버려짐으로써 첫번째 예외정보를 알 수 없었지만,  try-finally를 사용시에는 예외 하나가 보존되지만 숨겨진다. 그리고 stack trace 내역에 (suppressed)라는 라벨을 갖고 출력된다. 

```java
static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(
                new FileReader(path))){
            return br.readLine();
        } 
    }
```
