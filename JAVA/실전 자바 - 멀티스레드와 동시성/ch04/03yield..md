# yield

sleep과 yield의 차이를 알아본다.

```java
package thread.control.printer;

import static util.ThreadUtils.sleep;

public class YieldMainV2 {

    static final int THREAD_COUNT = 100;

    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            Thread thread = new Thread(new MyRunnable());
            thread.start();
        }
    }

    static class MyRunnable implements Runnable{
        @Override
        public void run() {
            for (int i = 0; i < 10; i++) {
                System.out.println(Thread.currentThread().getName() + " - " + i );
//                 sleep(1);
//                 Thread.yield();
            }
        }
    }

}

```

## sleep()

- sleep(1)을 사용한다면, 스레드의 상태를 1밀리초 동안 아주 잠깐 `RUNNABLE` -> `TIME_WAITING`으로 변경한다.
- 이렇게 되면 스레드는 CPU 자원을 사용하지 않고, 실행 스케줄링에서 잠시 제외(이탈)된다.
- 결과적으로 `TIMED_WATITING` 상태가 되면서 다른 스레드에 실행을 양보하게 된다. (대기열에 스레드가 없더라도 양보하게 된다.)
- `RUNNABLE` -> `TIMED_WAITING` -> `RUNNABLE` 과정을 거친다.

## yield()

자바 스레드가 `RUNNABLE` 상태 일 때, OS의 스케줄링은 다음과 같은 상태들을 가질 수 있다.
1. 실행 상태 (Running): 스레드가 CPU에서 실제로 수행 중
2. 실행 대기 상태(Ready): 스레드가 실행 준비는 되었으나, CPU가 바빠 스케줄링 큐에서 대기중이다.
자바는 두 상태를 구분할 수 없다. 둘 다 `RUNNABLE`이다.

### yield()의 작동
- Thread.yield() 메서드는 현재 실행 중인 스레드가 자발적으로 CPU를 양보하여, 다른 스레드가 실행될 수 있도록 한다.
- yield()는 RUNNABLE 상태를 유지하면서 CPU를 양보한다.
- 즉, `Running` 상태에서 스케줄링 큐에 다시 들어가는 `Ready` 상태로 들어가는 것이다.

- 자바에서 `Thread.yield()` 메서드를 호출하면, 현재 실행 중인 스레드가 CPU를 양보하도록 힌트를 준다.
- 자신에게 할당된 실행 시간을 포기하고 다른 스레드에게 실행 기회를 주도록한다.
- 힌트라고 표현하는 이유는 실행 순서를 강제로 양보하는 게 아니기 때문이다. -> 반드시 다른 스레드가 실행되는 건 아니다.
- 이것이, 예제에서 yield()를 사용할 때, 매번 다른 스레드가 실행되는 게 아니라, 몇개 겹치는게 보이는 이유이다.


## 프린터에서 활용하기

yield()를 적용하기 딱 좋은 곳이 있다.
바로 while 조건을 확인하는 곳이다. (CPU를 계속 사용한다.)

```java
while(!Thread.interrupted()){
   if(jobQueue.isEmpty()){
       continue;
    } 
}
```

이 코드는 쉴 틈 없이 CPU에서 이 로직을 반복해 수행한다. (1초에 while문을 수억번 반복할 수 도 있다.)

```java
while (!Thread.interrupted()) {
    if (jobQueue.isEmpty()) {
        Thread.yield();
        continue;
    }
}

```

위와 같이 인터럽트 상태가 아니라서, while문을 실행하기는 하지만, 결국 jobQueue가 비어있어서 아무 동작을 수행하지 않고 while문을 반복한다.
이때, Tehread.yield()를 사용한다면 다른 스레드에 양보할 수 있으니, 더 좋다는 의미이다.