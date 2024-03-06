## 아이템20: 추상 클래스보다는 인터페이스를 우선하라
- 자바가 제공하는 다중 구현 매커니즘: 인터페이스, 추상 클래스
- 자바8부터 인터페이스도 디폴트 메서드를 제공할 수 있어 인스턴스 메서드를 구현 형태로 제공할 수 있다

### 추상 클래스
- 추상 클래스를 구현하려면 반드시 추상 클래스의 하위 클래스가 되어야 한다.
- 다중 상속을 지원하지 않는 자바에는 커다란 제약사항(기존 클래에 추상 클래스 끼워넣기 어려움)

### 인터페이스
- 기존 클래스에 손쉽게 구현 가능
- 대상 클래스의 주된 기능에서 선택적 기능을 혼합(mixed in) 하는데 제격
- 구현 방법이 명확한게 있다면 디폴트 메서드로 제공 가능
  - 상속하려는 사람을 위한 설명을 @implSpec 자바독 태그 붙여서 문서화 필요(아이템19)
### 인터페이스의 장점을 활용한 패턴
#### 계층 구조가 없는 타입 프레임워크
- 현실에서 계층을 엄격히 구분하기 어려운 개념이 있다
``` java
// 가수
public interface Singer {
  AudioClip sing(Song s);
}
// 작곡가
public interface Songwriter {
  Song compose(int chartPosition);
}
// 가수와 작곡가는 부모-자식 계층이 아님. 작곡하는 가수
// 가수와, 작곡가 모두 구현 가능
// 새로운 메서드도 추가 가능
public interface SingerSongwriter extends Singer, Songwriter {
  AudioClip strum();
  void actSensitive();
}
```
#### 래퍼 클래스(아이템18 참조)
#### 템플릿 메서드 패턴(골격 구현 클래스)
- 인터페이스 + 추상 골격 구현 클래스
- 인터페이스와 추상클래스의 장점을 모두 취하는 방법
- 인터페이스로 타입을 정의 + default 메서드 제공
- 골격 구현 클래스는 나머지 메스드들을 구현
- 관례상 Abstract + Interface 이름 이름으로 지음
- 골격 구현을 확장하는 것으로 인터페이스 구현이 거의 끝남
``` java
// 코드 20-1 골격 구현을 사용해 완성한 구체 클래스 (133쪽)
static List<Integer> intArrayAsList(int[] a) {
  Objects.requireNonNull(a);

  // 다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능하다.
  // 더 낮은 버전을 사용한다면 <Integer>로 수정하자.
  // 익명 클래스(아이템24)
  return new AbstractList<>() {
      @Override public Integer get(int i) {
          return a[i];  // 오토박싱(아이템 6)
      }

      @Override public Integer set(int i, Integer val) {
          int oldVal = a[i];
          a[i] = val;     // 오토언박싱
          return oldVal;  // 오토박싱
      }

      @Override public int size() {
          return a.length;
      }
  };
}
```
- 예제
``` java
public interface Employee {
    public void attendance();
    public void work();
    public void leaveWork();
    public void earnMoney();
}
public class Developer implements Employee {

    @Override
    public void attendance() {
        System.out.println("출근!");
    }

    @Override
    public void work() {
        System.out.println("개발!");
    }

    @Override
    public void leaveWork() {
        System.out.println("퇴근!");
    }

    @Override
    public void earnMoney() {
        attendance();
        work();
        leaveWork();
    }
}
public class Manager implements Employee {

    @Override
    public void attendance() {
        System.out.println("출근!");
    }

    @Override
    public void work() {
        System.out.println("평가!");
    }
    
    @Override
    public void leaveWork() {
        System.out.println("퇴근!");
    }

    @Override
    public void earnMoney() {
        attendance();
        work();
        leaveWork();
    }
}
```
- 추상 골격 구현 클래스를 이용한 중복 코드 제거
``` java
public abstract class AbstractEmployee implements Employee {

    @Override
    public void attendance() {
        System.out.println("출근!");
    }
    
    @Override
    public void leaveWork() {
        System.out.println("퇴근!");
    }
    
    @Override
    public void earnMoney() {
        attendance();
        work();
        leaveWork();
    }
}
public class Developer extends AbstractEmployee {

    @Override
    public void work() {
        System.out.println("개발!");
    }
}
public class Manager extends AbstractEmployee {

    @Override
    public void work() {
        System.out.println("평가!");
    }
}
```
#### 시뮬레이트한 다중 상속(Simulated Multiple Inheritance)
  1. 구조상 골격 구현을 확장하지 못하면, 디폴트 메소드를 작성하고 직접 인터페이스를 구현
``` java
public interface Employee {
    void work();

    default void attendance() {
        System.out.println("출근!");
    }
    
    default void leaveWork() {
        System.out.println("퇴근!");
    }
       
    default void earnMoney() {
        attendance();
        work();
        leaveWork();
    }
}
```
  2. 인터페이스 구현 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달
``` java
// SalaryAccount를 이미 상속 받고 있기 때문에 골격 구현 클래스를 추가 상속할 수 없음
public class Developer extends SalaryAccount implements Employee {
    // 골격 클래스를 확장한 내부 클래스를 정의하고
    private DeveloperAbstractEmployee innerAbstractEmployee = new DeveloperAbstractEmployee();

    @Override
    public void attendance() {
        innerAbstractEmployee.attendance();  // 걱 메서드 호출을 내부 클래스 인스턴스에 전달
    }

    @Override
    public void work() {
        innerAbstractEmployee.work();
    }

    @Override
    public void leaveWork() {
        innerAbstractEmployee.leaveWork();
    }

    @Override
    public void earnMoney() {
        innerAbstractEmployee.earnMoney();
        salary();  // SalaryAccount 클래스의 메소드
    }
    
    private class DeveloperAbstractEmployee extends AbstractEmployee {
        @Override
        public void work() {
            System.out.println("개발!");
        }
    }
}
```
### 골격 구현 클래스 작성하는 법
- 인터페이스를 보며 다른 메서드들의 구현에 사용되는 기반 메서드 선정
- 기반 메서드들을 사용해 구현할 수 있는 메서드들을 디폴트 메서드로 제공
- 기반 메서드나 디폴트 메서드로 만들지 못한 메서드는 해당 인터페이스를 구현하는 골격 구현 클래스에서 작성
- 동작 방식을 잘 정리해 문서로 남겨야 한다
### 단순 구현(simple implementation)
- 골격 구현의 작은 변종
- 상속을 위해 인터페이스를 구현한 것이지만 추상 클래스가 아니다(AbstractMap.SimpleEntry 예제를 확인했는데 static 클래스인거 말고 일반 구현 클래스와 다른점이 무엇인지 모르겠음. 그냥 인터페이스 구현인데, 골격의 변종이라고 하는 사유가 뭔지?)
- 그대로 써도, 필요에 맞게 확장해서 써도 좋다
