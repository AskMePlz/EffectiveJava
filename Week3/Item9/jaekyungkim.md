# 아이템 9. try-finally 보다 try-with-resources를 사용하라

## 자원을 회수해야하는 객체

Java에서는 close 메서드를 호출 해, 직접 닫아줘야 하는 class들이 존재한다. 대표적으로 InputStream, OutputStream, java.sql.Connection등이 해당된다. 그래서 보통은
해당 자원을 닫기 위해서, try-finally를 가장 많이 사용한다.
<br>다음 예제 코드를 통해 다음과 같은 상황을 가정해보자.

```java

import java.io.BufferedReader;
import java.io.File;

public class item9Example {
    static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path));

        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }
}

```

이렇게 사용한 자원에 대해 닫기 위해서 try-finally를 사용한 것을 볼 수 있다. 하지만, 만약 자원을 하나 더 사용하면 어떻게 될까?

```java

import java.io.FileInputStream;

import java.io.FileOutputStream;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

class moreResourceExample {

    static void copy(String src, String dst) throws IOException {

        InputStream in = new FileInputStream(src);

        try {
            OutputStream out = new FileOutputStream(dst);

            try {
                byte[] buf = new byte[1024];
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
}

```

이렇게 작성 된 코드의 경우, 예외가 발생했을 때, 가장 첫 번째의 예외를 제대로 파악하기가 힘들다. try 문 finally문 모두 예외가 발생할 수 있어, 두번째로 발생한 예외가 첫 번째 예외를 삼키기 때문에
디버깅이 어려워지는 단점이 존재한다.

<br>
이런 문제를 해결하기 위해 java 7에서는 try-with-resources라는 것이 등장하였다. 

### try-with-resources, AutoCloseable 인터페이스 구현

첫번 째 코드를 바꿔보겠다.

```java
import java.io.BufferedReader;
import java.io.File;

public class item9Example {
    static String firstLineOfFile(String path) throws IOException {
        // BufferedReader br = new BufferedReader(new FileReader(path));
        //
        // try {
        //     return br.readLine();
        // } finally {
        //     br.close();
        // }

        try (BufferedReader br = new BufferedReader(
            new FileReader(path))) {
            return br.readLine();
        }

    }
}
```

다음은 두 번째 코드를 try-with-resources로 바꾼 예제이다.

```java

import java.io.FileInputStream;

import java.io.FileOutputStream;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

class moreResourceExample {

    static void copy(String src, String dst) throws IOException {
        //
        //     InputStream in = new FileInputStream(src);
        //
        //     try {
        //         OutputStream out = new FileOutputStream(dst);
        //
        //         try {
        //             byte[] buf = new byte[1024];
        //             int n;
        //             while ((n = in.read(buf)) >= 0)
        //                 out.write(buf, 0, n);
        //         } finally {
        //             out.close();
        //         }
        //     } finally {
        //         in.close();
        //     }
        //
        // }

        try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[1024];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }

}

```

이렇게 사용하면, 보다 더 코드를 간결하고 명확하게 사용이 가능하다. 심지어 try-with-resources는 catch절과도 함께 이용이 가능하다.

```java

import java.io.IOException;

class moreResourceExample {

    static void copy(String src, String dst) {

        try (BufferedReader br = new BufferedReader(
            new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            return dst;
        }
    }

}
```

이런식으로 catch절을 사용하면, try문을 중첩하지 않고도, 다수의 예외를 처리할 수 있다. 