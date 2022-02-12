### HashCode() 재정의 여부에 따른 결과값 비교 

``` java 

import java.util.HashMap;
import java.util.Objects;

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

    /*@Override
    public int hashCode() {
        return Objects.hash(name, age);
    }*/

    public static void main(String[] args) {
        User user1 = new User("yeonju", 20);
        User user2 = new User("yeonju", 20);
        System.out.println(user1.equals(user2)); // true

        HashMap<User, String> hashMap = new HashMap<>();
        hashMap.put(user1, "yj");

        System.out.println(hashMap.get(user1)); // yj 
        System.out.println(hashMap.get(user2)); // hash 재정의시 yj, 재정의 안했을 경우 null (기본 Object의 hashcode 사용) 
    }
}

``` 
