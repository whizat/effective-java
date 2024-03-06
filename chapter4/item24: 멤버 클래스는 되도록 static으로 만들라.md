# 아이템24: 멤버 클래스는 되도록 static으로 만들라
## 중첩클래스
- 다른 클래스 안에 정의되는 클래스
- 자신을 감싼 바깥 클래스에서만 쓰여야 함.
## 중첩 클래스 종류
- 정적 멤버 클래스
- (비정적) 멤버 클래스
- 익명 클래스
- 지역 클래스
## 정적 멤버 클래스
- 다른 클래스 안에 static으로 선언
- 바깥 클래스의 private 멤버에 접근 가능
- 외부 클래스와 독립적인 객체 생성 가능
- private 정적 멤버 클래스는 바깥 클래스가 표현하는 객체의 한 부분(구성요소)을 나타낼 때 쓴다
  - Map의 Entry
``` java
public class OuterClass {
    private String outerClassPrivateString;
    
    static class StaticMemberClass {
        void test() {
            OuterClass outerClass = new OuterClass();
            outerClass.outerClassPrivateString = "바깥 클래스 private 변수에 접근";
        }
    }

    public static void main(String[] args) {
        // 정적 멤버 클래스
        StaticMemberClass staticMemberClass = new StaticMemberClass();
    }
}
```
## 비정적 멤버 클래스
- 바깥 클래스 인스턴스 없이는 생성할 수 없다
  - 멤버 클래스의 인스턴스는 바깥 클래스의 인스턴스와 암묵적으로 연결된다
- 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들어야 한다
  - 숨은 외부 참조를 저장할 시간과 공간이 소모된다
  - GC가 바깥 클래스의 인스턴스를 수거하지 못해서 메모리 누수가 생길 수 있다
``` java
public class OuterClass {
    private String outerClassPrivateString;
    
    class MemberClass {
        void test() {
            OuterClass.this.outerClassPrivateString = "비정적 멤버 클래스도, 클래스명.this 형태로 바깥 클래스의 private 변수에 접근할 수 있다";  // 책에서 나온 표현. 바로도 접근 가능
        }
    }

    public static void main(String[] args) {
        OuterClass.MemberClass memberClass = new OuterClass().new MemberClass();
    }
}
```
- 보통 어댑터를 정의할 때 자주 쓰인다
  - 자신의 반복자를 구현할 때(예: Map, Set, List)
``` java
public class MySet<E> extends AbstractSet<E> {
  ...
  @Override
  public Iterator<E> iterator() {
    return new MyIterator();
  }

  private class MyIterator implements Iterator<E> {
    ...
  }
}
```
### 익명 클래스
- 이름이 없다
- 바깥 클래스의 멤버도 아니다
- 쓰이는 시점에 선언과 동시에 인스턴스가 만들어진다.
- 상수 변수 이외에 정적 멤버는 가질 수 없다.
- 정적 팩토리 메서드 구현(코드 20-1) 할 때 주로 쓴다
``` java
public class AnonymousClassExample {
    AnonymousInterfaceExample example = new AnonymousInterfaceExample() {
        @Override
        public void test() {
            System.out.println("익명 클래스");
        }
    };
}
interface AnonymousInterfaceExample {
    void test();
}
```
#### 익명 클래스 제약 사항
- 선언한 지점에만 인스턴스를 만들 수 있다.
- instanceof 검사나 클래스의 이름이 필요한 작업은 수행할 수 없다.
- 여러 인터페이스를 구현할 수 없다. 구현하는 동시에 다른 클래스를 상속할 수 없다.
- 익명 클래스를 사용하는 클라이언트는 그 익명 클래스가 상위 타입에서 상속한 멤버 외에는 호출할 수 없다.(?)
- 코드가 길어지면 가독성이 떨어진다.

### 지역 클래스
- 가장 드물게 사용
- 지역변수 선언할 수 있는곳이면 어디든 선언 가능하고, 유효 범위도 같다
- 멤버 클래스처럼 이름이 있고 반복해서 사용할 수 있다
- 익명 클래스처럼 비정적 문맥에서 사용될 때만 바깥 인스턴스를 참조할 수 있다
- 정적 멤버는 가질 수 없고, 가독성을 위해 짧게 작성해야 한다
``` java
public class LocalClassExample {
    
    private String value = "대충";

    void test() {
        class LocalClass {
            // 비정적 문맥에서만 바깥 인스턴스 참조 가능
            void print() {
                System.out.println(value);
            }
            // 정적 문맥에서는 바깥 인스턴스 참조 안됨 -> 컴파일 오류
            static void print2() {
                // System.out.println(value);
            }
        }
    }
}
```
## 정리
- 멤버 클래스의 인스턴스 각각이 바깥 인스턴스를 참조한다면 비정적 멤버 클래스, 그렇지 않다면 정적 멤버 클래스
- 중첩 클래스가 한 메서드 안에서만 쓰이면서 그 인스턴스를 생성하는 지점이 단 한 곳이고 해당 타입으로 쓰기에 적합한 클래스나 인터페이스가 이미 있다면 익명 클래스, 그렇지 않다면 지역 클래스
