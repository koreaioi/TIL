# 자바 메모리 모델 (Java Memory Model)

## Java Memory Model
Java Memory Model(JMM)은 자바 프로그램이 어떻게 메모리에 접근하고 수정할 수 있는지를 규정하며, 특히 멀티 스레드 프로그래밍에서 스레드간의 상호작용을 정의한다.   
JMM에는 여러 내용이 있지만, 핵심은 여러 스레드간의 작업 순서를 보장하는 `happens-before` 관계에 대한 정의다.

## happens-before
`happens-before` 관계는 자바 메모리 모델에서 스레드 간의 작업 순서를 정의하는 개념이다.   
만약 A작업이 B작업보다 happens-before 관계에 있다면, A작업에서의 모든 메모리 변경 사항은 B작업에서 볼 수 있다.   
즉, A작업에서 변경된 내용은 B작업이 시작되기 전에 모두 메모리에 반영된다는 뜻이다.

- happens-before 관계는 이름 그대로, 한 동작이 다른 동작보다 먼저 발생함을 보장한다.
- happens-before 관계는 스레드 간의 메모리 가시성을 보장하는 규칙이다.
- happens-before 관계가 성립하면, 한 스레드의 작업을 다른 스레드에서 볼 수 있게 되낟.
- 즉, 한 스레드에서 수행한 작업을 다른 스레드가참조할 때 최신 상태가 보장되는 것이다.
- 물론 항상 happens-before를 보장하는 건 아니고, volatile 같은 약속된 규율을 통해 보장된다.

