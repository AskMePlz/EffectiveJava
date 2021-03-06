# [item8] finalizer와 cleaner사용을 피하라



## finalizer 와 cleaner

이 두가지는 객체 소멸을 제공한다는 공통점이 있다. 

finalizer는 JVM에서 특정 인스턴스가 GC되어야 한다고 판단하고 메모리에서 지워지기 전에 finalize함수를 호출해 객체에서 사용하는 리소스를 해제한다. cleaner는 객체 참조 세트를 관리하고 청소한다. 

## finalizer의 부작용😱

- **finalizer와 cleaner는 즉시 수행된다는 보장이 없다.**
    
    자바 언어 명세는 finalizer나 cleanser의 수행시점 뿐 아니라 수행 여부조차 보장하지 않기 때문에 언제 수행할지 모른다. 때문에 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다. 
    
    예)  시스템은 동시에 열수 있는 파일 개수에 한계가 있는데, finalizer 또는 cleaner가 실행을 게을리해서 파일을 계속 열어두면 새로운 파일을 여지 못해 프로그램이 실패할 수 있다.
    

- **finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.**
    
    해당 객체가 마무리가 덜 된 채로 남을 수 있다.(훼손될 수 있다.)
    
    이렇게 훼손된 채로 남은 객체를 다른 쓰레드는 훼손이 되었는지 알지 못한다. 때문에 모르고 사용했다가 예상치 못한 오류를 마주할 수 있다.
    
- **finalizer와 cleaner의 성능 문제**
    
    성능이 많이 떨어진다.
    
- **finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.**
    
    생성자나 직렬화 과정에서 예외가 발생하면 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다. 이 finalizer는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다. 
    

## finalizer나 cleaner를 대신해줄 묘안은 없을까

 AutoCloseable를 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면된다.
