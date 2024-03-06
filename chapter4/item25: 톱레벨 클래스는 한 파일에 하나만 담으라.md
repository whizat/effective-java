# 아이템: 톱레벨 클래스는 한 파일에 하나만 담으라
- 소스 파일에 톱레벨 클래스 여러개 선언해도 컴파일 에러는 안남
- 근데 쓰지 마셈
  - 클래스의 의미를 여러가지로 정의할 수 있음
  - 그중 어느것을 사용할 지는 어느 소스 파일이 먼저 컴파일하냐에 따라서 달라짐
## 두 클래스가 한 파일에 정의된 경우
- Main.java
``` java
// (150쪽)
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```
- Utensil.java
``` java
// 코드 25-1 두 클래스가 한 파일(Utensil.java)에 정의되었다. - 따라 하지 말 것! (150쪽)
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```
  - Dessert.java
``` java
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```
- javac Main.java Dessert.java
  - Utensil, Dessert 클래스 중복 정의 컴파일 오류
  - Main -> Utensil -> Dessert -> Dessert(클래스 중복 정의 컴파일 오류)
- javac Main.java, javac Main.java Utensil.java
  - pancake
- javac Dessert.java Main.java
  - potpie
 
## 해결책
- 톱레벨 클래스들을 서로 다른 소스 파일로 분리
- 굳이 한 파일에 담고 싶으면 정적 멤버 클래스(아이템24) 사용
  - 다른 클래스에 딸린 부차적인 클래스라면 이게 더 나음
  - private으로 선언하면 접근 범위도 최소로 관리 가능
``` java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```
## 정리
- 소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자
