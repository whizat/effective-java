## 아이템17: 변경 가능성을 최소화하라
불변 클래스는 간단히 말해 인스턴스 내부값을 수정할 수 없는 클래스. 객체가 파괴되는 순간까지 달라지지 않음
가변 클래스보다 설계, 구현, 사용 쉬움. 오류 생길 가능성도 적고, 안전
### 불변 클래스 만드는 5가지 규칙
- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
- 클래스를 확장할 수 없도록 한다.
- 모든 필드를 final로 선언한다.
- 모든 필드를 private로 선언한다. 
- 자신 외에는 내부 가변 컴포넌트에 접근할 수 없도록 한다.(방어적 복사)
  - 생성자, 접근자 readObject 메서드(아이템 88) 모두에서 방어적 복사 수행

#### 불변 복소수 클래스 예제
``` java
public final class Complex {
    private final double re;
    private final double im;

    public static final Complex ZERO = new Complex(0, 0);
    public static final Complex ONE  = new Complex(1, 0);
    public static final Complex I    = new Complex(0, 1);

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart()      { return re; }
    public double imaginaryPart() { return im; }
    // 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어서 반환(함수형 프로그래밍)
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;

        // == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
#### 불변 클래스 장점
- 함수형 프로그래밍: 피연산자에 함수를 적용해 결과를 반환하지만, 피연산자 자체는 그대로인 프로그램
  - 메서드 이름으로 동사(add)가 아닌 전치사(plus)를 사용해야 사람들이 함수형인지 명령형인지 대충 알아먹고 씀
  - 객체를 변경하는것이 아니기 때문에 불변이 되는 영역의 비율을 높임
- 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요가 없다.
  - 안심하고 공유 가능해서 최대한 재활용하는게 좋음.(예: 상수)
- 불변 클래스에서 자주 사용되는 인스턴스를 캐싱하여 인스턴스를 중복 생성하지 않게 해주는 정적 팩토리 메소드 제공 가능
  - BigInteger static, valueOf 참조
- 복사해도 원본과 똑같으므로, 방어적 복사도 필요 없다
- 불변 객체끼리는 내부 데이터를 공유할 수 있다.
  - BigInteger negate 함수 참조
- 객체를 만들 때 불변 객체들을 구성요소로 사용하면 이점이 많다.
  - Map의 Key와 Set의 원소로 쓰기 좋음
- 불변 객체는 그 자체로 실패 원자성을 제공한다(아이템 76)
  - 실패 원자성: 메서드에서 예외가 발생한 후에도 그 객체는 여전히 (메서드 호출 전과 똑같은) 유효한 상태여야 한다
  - 메서드 예외 발생하고, catch로 잡은 후에도 객체값 안 변함. 클라이언트에서 계속 재사용 가능
#### 불변 클래스 단점
- 값이 다르면 반드시 독립적인 객체로 만들어야 함
  - 값이 많으면 비용이 비쌈
- 원하는 객체를 완성하기까지 단계가 많고, 그 중간 단게에 만든 객체들이 모두 버려진다면 성능 이슈
##### 문제 해결 방법
1. 다단계 연산들을 예측해 기본 기능으로 제공
   - 각 단계마다 객체 생성 안해도 됨
   - 불면 필드로 오래걸리는 연산을 가변 필드로 처리하기 위해서 package-private 가변 클래스를 만들어서 외부에 API로 노출 하지 않고 불변 클래스 내에서 사용
   - 예) BigInteger와 MutableBigInteger
2. 예측하기 어렵다면 package-private의 가변 동반 클래스로 제공
   - String, StringBuilder, StringBuffer
### 불변 클래스로 만드는 방법
1. class를 final로 선언
2. 모든 생성자 private 혹은 package-private 선언 후, public 정적 팩터리 제공(아이템1)
``` java
public static Complex valueOf(double re, double im) {
    return new Complex(re, im);
}
```
### 정리
- Getter가 있다고 해서 무조건 Setter를 구현하지 말자.
- 클래스는 꼭 필요한 경우가 아니라면 불변이여야한다.
- 불변으로 만들 수 없는 클래스이더라도 변경할 수 있는 부분은 최대한 줄이자.
  - 다른 합당한 이유가 없다면 모든 필드는 private final 이여야한다.
- 생성자는 불변식 설정이 모두 완료된, 객체를 생성해야한다.
  - 확실한 이유가 없다면 생성자와 정적 팩터리 이외에는 그 어떤 초기화 메서드도 public으로 제공하면 안된다.

불변식의 원칙을 잘 지킨 클래스 예제 CountDownLatch
