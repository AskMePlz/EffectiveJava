
### equals 재정의 안한 경우 false -> 객체 식별, 동일성 O, 주소값 비교 
``` java
public class User {

    private String name;
    private Integer age;

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }


    public static void main(String[] args) {
        User user1 = new User("yeonju", 20);
        User user2 = new User("yeonju", 20);
        System.out.println(user1.equals(user2)); // false 
    }
}
```
``` java 
// Object에 equals 적용 
public boolean equals(Object obj) {
    return (this == obj);
}
```     

### equals 재정의 한 경우 true -> 논리적 동치성, 동등성, 값 비교로 재정의 
``` java
public class User {

    private String name;
    private Integer age;

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return Objects.equals(name, user.name) &&
                Objects.equals(age, user.age);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    public static void main(String[] args) {
        User user1 = new User("yeonju", 20);
        User user2 = new User("yeonju", 20);
        System.out.println(user1.equals(user2)); // true 
    }
}
```

### String equals true -> String class에 재정의 되어 있음 

``` java 
String a = "a";
String b = "b";
System.out.println(a.equals(b));  // true
```

``` java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

   
