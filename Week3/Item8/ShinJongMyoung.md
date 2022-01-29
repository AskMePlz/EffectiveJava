# Item8. finalizer 와 cleaner 사용을 피하라

### finalize 와 cleaner 의 문제점
- 언제 호출될지 예측할 수 없다.
- 막상 호출됐을 때, 상황에 따라 위험할 수 있다.
- 성능 문제
- 보안 문제

### finalizer 와 cleaner 를 대신해 줄 묘안
- 그저 `AutoCloseable` 를 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다.

### 그렇다면 언제 finalize 와 cleaner 를 사용해야 할까?
1. 자원의 소유자가 close 메서드를 미처 호출하지 않는 것에 대비한 안전망
2. 네이티브 피어와 연결된 객체에 사용
