@ public static void main 인 이유

# 서론
자바 프로그램은 자바 가상 머신 (JVM: Java Virtual Machine)을 통해 실행된다.

JVM은 main 이라는 이름이 붙은 메서드를 찾아 프로그램을 시작하도록 설계 되어있다. (개발자들의 약속)   
이 때, JVM이 main 메서드를 올바르게 식별하고 실행하도록 알맞은 제어자modifier가 부착되어야한다.

# 1. public
public은 접근 제어자 중 하나이다.   
public을 사용해, 접근 제한이 없고 자바 프로젝트 어디서든지 자유롭게 접근할 수 있도록한다.

JVM이 Java를 시작한 다음, main 메서드를 실행하려는 시점은 어떤 클래스도 Load 되어있지 않다.   
가장 먼저 main 메서드의 위치를 찾아 해당 클래스를 불러온다.   
이때 main에 접근 제약이 없도록 public 접근 제어자를 사용한다.

- public : 접근 제한 없다.
- protected : 같은 패키지 내 + 다른 패키지의 자식 클래스
- default : 같은 패키지 내에서 접근 가능
- private : 같은 클래스 (같은 패키지의)에서만 접근 가능


# 2. static

static을 사용하면 Java가 컴파일 되는 시점에 정적 멤버(변수)를 메모리에 할당해, 인스턴스 없이 접근할 수 있도록 한다.   
JVM이 프로그램을 시작할 때는 어떤 객체도 생성하지 않기 때문에, 인스턴스 없이 접근할 수 있는 static이 필요하다.


# 3. void
main 메서드가 종료되면 그대로 프로그램도 종료된다.      
따라서 main 메서드의 반환값이 필요로하지 않다.

결국 main 메서드의 반환타입은 void이다.

# 4. 매개변수 String[] args

The Java Virtual Machine starts execution by invoking the method main of some specified class, passing it a single argument, which is an array of strings

자바 가상 머신(JVM)은 특정 클래스의 메인 메서드를 호출함으로써 프로그램 실행을 시작하는데,
이 때 String 배열 타입의 인자 하나를 전달합니다.

─ Java Language Specification, 12.1. [Java Virtual Machine Startup]

#### 명령 프롬프트로의 실행 예시
![image](https://github.com/user-attachments/assets/76e0b7aa-84e6-448e-8e84-ec3810b3ce2f)

javac Example.java -encodig UTF-8   
-> java 파일을 JVM이 이해할 수 있도록 .class파일로 변환한다. (컴파일)

java test.Example aa bb cc
-> test폴더에 있는 Example.class를 실행하고, main함수에 aa, bb, cc를 String 배열로 전달한다.   
args에 aa, bb, cc가 전달된다.

#### IDE 실행
intelliJ 혹은 Eclipse 같은 IDE에서도 각각 설정을 통해 main 메서드에 String 배열 인자를 전달할 수 있다
