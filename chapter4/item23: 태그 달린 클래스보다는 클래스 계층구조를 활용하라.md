# 아이템23: 태그 달린 클래스보다는 클래스 계층구조를 활용하라
## 태그 달린 클래스
``` java
// 코드 23-1 태그 달린 클래스 - 클래스 계층구조보다 훨씬 나쁘다! (142-143쪽)
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```
### 단점
- 열거 타입, 태그 필드, switch문 등 쓸데 없는 코드가 많아 가독성 나쁨
- 다른 의미를 위한 코드가 언제나 존재해서 메모리 많이 사용
- 필드들을 final로 선언하려면 해당 의미에서 쓰이지 않는 필드까지 생성자 초기화 필요
  - 쓰지 않는 필드 초기화하는 불필요한 코드가 늘어남
  - 엉뚱한 필드를 초기화 해도 컴파일러가 도와주지 못함. 런타임에 문제 들어남
- 새로운 의미를 추가할 때마다 작성해야 하는 코드가 많음
- 인스턴스의 타입만으로는 현재 나타내는 의미를 알 수 없음
**태그 달린 클래스는 장황하고, 오류를 내기 쉽고, 비효율적**

## 클래스 계층 구조를 활용한 서브타이핑(subtyping)
``` java
abstract class Figure {
    abstract double area(); 
}

class Circle extends Figure {
    final double radius;
    
    Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;
    
    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    @Override
    double area() {
        return length * width;
    }
}
```
- 태그 달린 클래스의 단점 모두 해결!
### 정리
**태그 달린 클래스 쓰지마!!**
