#### 아이템8
# finalizer와 cleaner 사용을 피하라

###  자바의 두 가지 객체 소멸자 - finalizer, cleaner
- finalizer : 예측할 수 없고, 상황에 따라 위험할 수도 있어 일반적으로 불필요 -> 오동작, 낮은성능, 이식성 문제의 원인이 되기도 함
   - 기본적으로 **쓰지말아야'** 한다.
- cleaner : finalizer 보다 덜 위험하지만 여전히 예측불가, 느리고 일반적으로 불필요
   - 자바9에서 finalizer를 사용자제(deprecated) API 지정, cleaner를 대안으로 소개

### finalizer와 cleaner는 C++ distructor(파괴자)와 다른 개념
- 생성자의 꼭 필요한 대척점으로 특정 객체와 관련된 자원을 회수하는 보편적인 방법
- 자바에서는 접근할 수 없게 된 객체를 회수하는 역할을 가비지 컬렉터가 담당, 프로그래머에게 아무런 작업도 요구하지 않음
- C++의 distructor는 비메모리 자원 회수용도로 쓰이나 자바에서는 try-with-resources, try-finally를 사용하여 해결(아이템 9)

### finalizer와 cleaner의 문제점
#### 즉시 수행된다는 보장이 없다
- 객체에 접근할 수 없게 된 후 해당 소멸자들이 실행되기까지 얼마나 걸리는지 알수 없어 제 떄 실행되어야 하는 작업은 절대 할 수 없다
- 해당 소멸자들이 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달렸으며 이는 가비지 컬렉터 구현마다 천차만별이다
   - 여러분이 테스트한 JVM에서는 완벽하게 동작하던 프로그램이 고객의 시스템에서는 엄청난 재앙을 일으킬지도 모르다.
#### 수행지점 뿐 아니라 수행여부도 보장하지 않는다.
- 접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한 채 프로그램이 중단될 수 있음
   - 상태를 영구적으로 수정하는 작업에서는 절때 의존해서는 안됨.
   - 예를들어 데이터베이스 같은 공유 자원의 영구 락(lock) 해제를 맡겨 놓으면 분산 시스템 전체가 서서히 멈출 것이다.
#### System.gc, System.runFinalization 메서드에 현혹되지 말자.
- 실행될 가능성을 높여줄 수 있으나 보장해주지 않음.
#### finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.
- 잡지 못한 예외 때문에 해당 객체는 마무리가 덜 된 상태로 남을 수 있으며 다른 스레드가 훼손된 객체를 사용하려고 한다면 어떻게 동작할지 예측 불가
- cleaner를 사용하는 라이브러리는 자신의 스레드를 통제하기 떄문에 이러한 문제가 발생하지 않음.
#### 심각한 성능 문제 동반
#### finalizer를 사용한 클래스는 finalizer 공격에 노출, 심각한 보안문제를 일으킬 수 있다.
- 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다만 객체에서 악의적인 하위 클래스의 finalizer가 수행될수 있게 된다. 이 finalizer는 정적 필드에 자신의 참조를 할당, 가비지 컬렉터가 수집못하게 막을 수 있으며 이 객체의 메서드를 호출해 애초에는 허용되지 않았을 작업을 수행하는 건 일도 아니다.
- final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무일도 하지 않는 finalize 메서드를 만들고 final로 선언하자. 

### finalizer, cleaner를 대신해줄 묘안은?
>> AutoCloseable을 구현해주고
>> 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다
>> 일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야 한다

### finalizer, cleaner의 적절한 쓰임새는?
- 자원의 소유자가 close 메서드를 호출하지 않을 것을 대비한 안전망 역할
- 네이티브 피어(일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체)와 연결된 객체
   - 네이티브 피어는 자바 객체가 아니니 GC는 그 존재를 알지 못해 자바 피어를 회수할 때 네이티브 객체까지 회수하지 못하기 때문에 clenaer, finalizer가 나서서 처리하기에 적당한 작업
   - 단, 성능 저하를 감당할 수 없거나 네이티브 피어가 사용한 자원을 즉시 회수해야 한다면 앞서 설명한 close 메서드 사용

-----
>> cleaner( 자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자
>> 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다
-----

##### 참고
finalize() - 무순서 실행 예시 https://seeminglyjs.tistory.com/193
