## 아이템16: public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

### public 필드를 가지고 있는 클래스
``` java
class Point {
    public double x;
    public double y;
}
```
- 캡슐화 이점 제공 X
- API 수정하지 않고 내부 표현 못바꿈
- 불변식 보장 X
- 외부에서 필드에 접근할 때 부수 작업 수행 X
### 필드를 모두 private으로 바꾸고 public 접근자(getter) 추가
``` java
class Point{
    private double x;
    private double y;

    public Point(double x, double y){
        this.x = x;
        this.y = y;
    }

    public double getX(){
        return x;
    }
    public double getY(){
        return y;
    }
    public void setX(double x){
        return this.x = x;
    }
    public void setY(double y){
        return this.y = y;
    }
}
```
public 클래스라면 패키지 바깥에서 접근할 수 있는 접근자를 제공함으로써 내부 표현 방식을 맘대로 바꿀 수 있는 유연성을 얻을 수 있음

**하지만 private-package 클래스 혹은 private 중첩 클래스는 데이터 필드를 노출해도 됨**
- 패키지 바깥 코드는 접근할 수 없다.
- prviate 중첩 클래스는 해당 클래스를 포함하는 외부 클래스까지만 접근 가능

public 클래스의 public 데이터 필드가 불변이라면, 불변식 보장 외에 public 데이터 필드의 단점과 똑같다.
