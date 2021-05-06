# HashMap

- **해시 테이블 기반** Map 인터페이스 구현체
- [Java Collections Framework](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html)의 멤버
- 모든 옵션의 Map 연산을 제공하며 **key와 value에 `null`를 허용**
  - null을 허용하는 것과 비동기적이라는 것을 제외하면 [Hashtable](https://docs.oracle.com/javase/8/docs/api/java/util/Hashtable.html)과 거의 유사함
- Map의 **순서를 보장하지 않음** 
  - 특히, 시간이 지남에 따라 순서가 일정하게 유지됨을 보장하지 못함
  - 해시 함수가 버킷 간에 원소들을 적당히 흩어놓는다는 가정하에 get, put과 같은 기본 연산에 상수 시간의 성능을 제공

## HashMap 인스턴스의 성능에 영향을 주는 요인: 초기 용량과 부하율

### 용량과 초기 용량
- `용량` *capacity* : 해시 테이블 내부 버킷의 수
- `초기 용량` *initial capacity* : 해시 테이블이 생성되는 시점의 용량

### 부하율
- *Load Factor* : 해시 테이블의 용량이 자동으로 증가하기 전에 얼마나 꽉 채울지에 대한 척도를 의미
- `내부 entry의 수`가 `부하율과 현재 용량의 곱`을 초과하게 되면, 해시 테이블은 버킷의 수가 약 두 배가 되도록 **재해싱** 
  - 즉, **내부 데이터 구조가 다시 빌드된다**
- 일반적으로, 기본 부하율(.75)은 시간과 공간 비용 간 적절한 균형 *tradeoff*을 제공
  - 더 높은 값은 공간 오버헤드를 줄이지만 조회 비용이 증가
  - get, put을 포함한 HashMapp 클래스의 대부분의 연산에 반영
- 재해시 작업의 수를 최소화하기 위해, 초기 용량 설정시 map 내부에 예상되는 entry의 수와 부하율을 고려해야 함

### 성능이 중요하다면
- 많은 매핑이 인스턴스에 저장되어야 한다면, 미리 충분히 큰 용량으로 생성하라
  - 재해싱 오버헤드 > 초기화 오버헤드
- 동일한 hashCode()를 갖는 많은 수의 key를 사용하는 것은 해시 테이블의 성능을 낮추므로 주의할 것

### 비동기 unsynchronized
- **HashMap은 동기화되지 않는 구현** 
- 만약 다수의 스레드가 동시에 Map에 접근하고 적어도 하나의 스레드가 Map을 **구조적으로 변경**한다면, 이는 외부적으로 동기화되어야 함
  - **구조적 변경이란 하나 이상의 매핑을 추가하거나 제거하는 연산**을 의미
    - 단순히 인스턴스에 포함된 key의 value를 변경하는 것은 구조적 변경이 아니다!
- 일반적으로 Map을 자연스럽게 캡슐화하는 객체를 동기화함으로써 수행
  - 만약 그런 객체가 존재하지 않는다면, Map은 `Collections.synchronizedMap` 메소드를 이용해 wrapped 되어야 함
  - 비동기화되고 예상치 못한 접근을 방지하기 위해 Map 생성시 이를 수행하는 것이 일반적

```java
Map m = Collections.synchronizedMap(new HashMap(...));
```

## Collection view iteration
- HashMap 인스턴스의 **용량** *capacity(버킷의 수)* 와, **크기** *size(key-value 매핑 수)* 에 비례한 시간이 필요
  - 따라서, iteration 성능이 중요한 경우에는 
    - 초기 용량을 너무 크게 잡지 않고
    - 부하율을 너무 낮게 설정하지 말아야 함
- **빠른 실패 fail-fast**
  - 만일 iterator가 생성된 이후 어느 때나 *(iterator 자체 remove()는 제외하고)* 어떤 방식으로든 Map이 구조적으로 변경될 때, iterator는 [ConcurrentModificationException](https://docs.oracle.com/javase/8/docs/api/java/util/ConcurrentModificationException.html)을 발생시킴
  - 즉, iterator는 동시 변경을 직면하면 위험을 감수하기보다는 빠르고 깔끔하게 실패한다
  - 일반적으로는 동기화되지 않은 동시 변경이 있는 경우 어떠한 엄격한 보장도 할 수 없기 때문에, iterator의 fail-fast 동작이 보장될 수 없음
    - 그러므로 이 예외에 의존하는 프로그램을 작성하는 것은 정확도가 잘못될 수 있음
    - iterator의 fail-fast 행동은 오직 버그를 감지하기 위해 사용되어져야 한다

Reference. [HashMap Doc](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html)