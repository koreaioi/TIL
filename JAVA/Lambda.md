# 0. 자바의 정석을 다시 공부하는 이유
   TAVE 후반기 팀 프로젝트를 하면서 자바 문법에 대해서 부족함을 많이 느꼈다.
1. 예외처리에 대해서 부족함을 느꼈다.
2. 람다식과 스트림에 대해서 부족함을 느꼈다.

따라서 자바의 정석 2를 다시 공부하자   

# 1. 람다식(Lambda expression)
람다식 = 함수를 간단한 식(expression)으로 표현하는 방법
이런 식을

```java
    int max(int a, int b){
        return a > b ? a : b;
    }
```

아래와 같이 표현하는 걸 람다식이라고 한다.

```java
    (a,b) -> a > b ? a : b
```


람다식은 이름이 없는 익명함수이다.(엄밀히 말하면 익명 객체이다.)



### 함수와 메서드의 차이

1. 함수는 일반적 용어, 메서드는 객체지향개념 용어

2. 함수는 클래스에 독립적, 메서드는 클래스에 종속적이다.



### 람다식 만들기

1. 반환 타입과 이름을 지운다.

2. 소괄호와 중괄호 사이에 화살표 ( -> )를 추가한다.

3. 반환값이 있는 경우, 식이나 값만 적고 return문을 생략할 수 있다. + 세미 콜론 생략

4. 매개변수의 타입이 추론 가능하면 생략한다. (대부분 생략가능하다)



### 람다식 주의사항

1. 매개변수가 하나면 소괄호 () 생략 가능

```java
(a) -> a * a
        
a -> a * a // 괄호 생략 가능
int a -> a * a // error, 변수 타입 없어야 함
```

2. 블록 안의 문장이 하나면 중괄호 {} 생략 가능

```java
(int i) -> {
    System.out.println(i);
}

// 람다식
(int i) -> System.out.println(i)
```




# 2. 람다식 예
   다음 메서드를 람다식으로 바꿔보자.

#### 함수 예시
```java
// 1.
int max(int a, int b){
    return a > b ? a : b;
}
// 2.
int printVar(String name, inti){
    System.out.println(name+"="+i);
}
// 3.
int sqaure(int x){
    return x * x;
}
// 4.
int roll(){
    return (int)(Math.random() * 6);
}

```

#### 람다식으로 바꾸기

```java

// 1.
(a,b) -> a > b ? a : b;
// 2.
(name, i) -> System.out.println(name+"="+i)
// 3.
x -> x * x
// 4.
() -> (int)(Math.random() * 6)

```


# 3. 람다식은 익명 객체!
참고 - 익명 클래스 (7장)   
객체의 선언과 생성을 동시에한다.

```java

// 이 식은
(a, b) -> a > b ? a: b;

// 아래의 익명 객체이다.

new Object() {
    int max(int a, int b){
      return a > b ? a : b;
    }
}

// 람다식(익명 객체)를 다루기 위한 참조 변수가 필요한데
// 이때, 참조변수의 타입은? - Object??

Object obj = new Object() {
    int max(int a, int b){
        return a > b ? a : b;
    }
}

```


### Object로 할 때 문제점.  
Ojbect 타입에는 내가 만든 메서드가 없어 obj.max()와 같은 메서드를 사용할 수 없음
이 문제 해결을 위해 함수형 인터페이스를 사용한다.



# 4. 함수형 인터페이스
   함수형 인터페이스 - 단 하나의 추상 메서드만 선언된 인터페이스

```java

// 단 하나의 추상 메서드만 선언된 함수형 인터페이스
interface MyFunction {
    public abstract int max(int a, int b);
}

// 익명 클래스로 함수형 인터페이스를 사용
MyFunction f = new MyFunction() {
    @Override
        public int max(int a, int b) {
            return a > b ? a : b;
    }
}



// 위 방식을
// 함수형 인터페이스의 참조변수로 람다식을 참조한다.
// 함수형 인터페이스의 메서드와 람다식의 매개변수 개수 + 반환 타입이 일치해야함.
MyFunction f = (a,b) -> a > b ? a : b;
int value = f.max(3,5);

```


### 함수형 인터페이스의 사용 예시

컬렉션의 sort 함수 원래면 아래와 같이 작성

```java

List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");

Collections.sort(list, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s2.compareTo(s1);
    }
});

```


하지만 Comparator는 아래와 같이 @FunctionalInterface가 붙어있는 함수형 인터페이스 이다.


따라서 이를 람다식으로 바꾼다.

1. 함수명과 반환타입 지우고 변수 타입 지운다.

결과

List<String> list = Arrays.asList("abc", "aaa", "bbb", "ddd", "aaa");
// 람다식!
Collections.sort(list, (s1, s2) -> s2.compareTo(s1));




# 5. 함수형 인터페이스 타입의 매개변수, 반환타입
아래와 같은 MyFunction이라는 함수형 인터페이스가 있고 메서드 이름은 myMethod이다.

```java

@FunctionalInterface
interface MyFunction{
    void myMethod();
}

```

aMethod라는 함수의 역할은 MyFunction 함수형 인터페이스 타입을 매개변수로 받아서 MyFunction의 유일한 메서드인 myMethod를 실행한다.

```java
void aMethod(MyFunction f){
    f.myMethod();
}


```

따라서 위 두가지를 활용하는 코드 예시는 아래와 같다.

```java

MyFunction f = () -> System.out.println("myMethod()");
aMethod(f);
```

즉 () - > System.out.println("myMethod()"); 부분이 MyFunction 함수형인터페이스의 myMethod가 되는 것.

그리고 aMethod라는 메서드는 f의 myMethod를 실행한다.



이를 최종적으로 줄이면

```java
aMethod(() -> System.out.println("myMethod()"));
```

가 된다.
