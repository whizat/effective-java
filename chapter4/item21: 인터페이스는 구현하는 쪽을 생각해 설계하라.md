## 아이템: 인터페이스는 구현하는 쪽을 생각해 설계하라
- 자바8 이후에는 디폴트 메서드를 제공하여, 구현체를 깨뜨리지 않고 인터페이스에 메서드를 추가할 방법이 생김
- 하지만 기존 구현체들과 매끄럽게 연동되리란 보장 X

### removeIf
- predicate가 true를 반환 하면 반복자의 remove 메서드를 호출해서 그 원소 제거
``` java
default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
	boolean removed = false;
	for (Iterator<E> it = iterator(); it.hasNext(); ) {
        if (filter.test(it.next())) {
            it.remove();
            result = true;
        }
    }
	return result;
}
```
- apache의 SynchronizedCollection에 4.4 버전 이전에서는 removeIf를 재정의하지 않았다(4.4 버전 이후로는 재정의 함)
- 디폴트메서드인 removeIf를 호출하게 된다면, 동기화해주지 못한다. -> 멀티쓰레드 환경에서 ConcurrentModificationException이 발생하거나, 예기치 못한 결과로 이어질 수 있다
- 참고: https://commons.apache.org/proper/commons-collections/apidocs/src-html/org/apache/commons/collections4/collection/SynchronizedCollection.html#line.193

### 결론
- 디폴트 메서드는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있다.
- 디폴트 메서드라는 도구가 생겼더라고 인터페이스를 설계할 때는 여전히 세심한 주의를 기울여야 한다.
