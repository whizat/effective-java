## 아이템22: 인터페이스는 타입을 정의하는 용도로만 사용하라
- 인터페이스를 구현한다는것은, 구현 클래스의 인스턴스가 무엇을 할 수 있는지를 클라이언트에 얘기해주는 것.

### 상수 인터페이스(안티패턴-사용금지!)
- 상수를 뜻하는 static final 필드로만 가득 찬 인터페이스
``` java
// 코드 22-1 상수 인터페이스 안티패턴 - 사용금지! (139쪽)
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```
### 문제점
- 내부 구현을 클래스의 API로 노출하는 행위
- 클라이언트 코드가 내부 구현에 해당하는 이 상수들에 종속된다

### 상수 클래스의 몇가지 좋은 예
- 클래스 자체에 추가
  - Integer.MIN_VALUE
- Enum 타입으로 생성(아이템 34)
- 인스턴스화 할 수 없는 유틸리티 클래스(아이템 4)에 담아 공개
``` java
// 코드 22-2 상수 유틸리티 클래스 (140쪽)
public class PhysicalConstants {
  private PhysicalConstants() { }  // 인스턴스화 방지

  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  // 볼츠만 상수 (J/K)
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

  // 전자 질량 (kg)
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```
