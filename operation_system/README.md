### 1. Process와 Thread의 차이점을 설명해보세요

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

Process는 memory 상에 존재하는 **실행 가능한 프로그램** 이고, Thread는 **Process로부터 자원을 할당 받아서 Process와 code(text), heap, stack 영역을 공유합니다**

* 쓰레드는 Context switching 과정에서 프로세스와는 다르게 캐시 메모리를 비우지 않아도 되기 때문에 context switching 비용이 낮습니다.
  (프로세스의 경우 캐시 메모리를 모두 비우고 이에 대한 정보를 모두 PCB에 저장합니다.)
* 쓰레드는 Process와는 다르게 CPU 자원을 주로 할당받는다는 것도 차이점이 존재합니다. (Windows 기준임. 다른 OS는 잘 모르겠음.)

</div>
</details>

### 2. Context Switching에 관해서도 설명해주실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

CPU는 1개의 코어당 1개의 프로세스만을 실행할 수 있습니다. 따라서 OS는 프로세스의 동시성을 위해서 Process를 특정 알고리즘을 이용해서 CPU에 할당했다가 해제시키는 행위를 반복합니다.

이 때 사용하는 알고리즘을 **Process Scheduling 알고리즘** 이라고 부르고, CPU에 프로세스가 할당되는 과정을 **Dispatch** 라고 부릅니다.

Process는 문맥 교체 과정에서 캐시 메모리를 모두 비우고 PCB에 해당 정보를 모두 저장하는데, 이 때 저장되는 정보는 Register에 대한 정보, PC(Program Counter) 등이 있습니다. 그리고 CPU에 다시 dispatch가 되는 시점에 PCB를 읽어서
 프로세스의 중단 지점에 대한 데이터를 모두 회복합니다. 이를 통해 Process를 중단 지점에서 다시 시작할 수 있게됩니다.

</div>
</details>

### 3. 동기 처리, 비동기 처리에 대해서 설명해주실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

동기 처리는 메소드의 리턴 시점과 결과 전달 시점이 일치하는 처리 방식이고, 비동기 처리 방식은 작업의 완료 여부와 관계없이 로직이 실행되는 처리 방식입니다.

동기 처리의 경우 결과에 대해서 높은 신뢰도를 가진다는 장점이 있으나, 처리 속도가 늦다는 단점이 있고, 비동기 처리의 경우 단위 시간당 처리량이 높다는 장점 대신에 데이터에 대한 일관성이 헤쳐질 가능성이 존재합니다.

> 👉 **이는 Kafka Consumer에서도 동일한 내용이 등장합니다. consumer의 commit에 대해서 데이터의 순서성이 무조건적으로 보장되어야하는 환경이면 commitSync()를 호출해야하고,** 
> **그렇지 않고 처리 속도만이 중요하다고 하면 commitAsync()를 이용해서 커밋 처리를 수행해야합니다.** 
> **Kafka의 경우 default commit 정책이 commitSync()이기 때문에 비동기적인 커밋을 위해선 Auto Commit을 false로 설정하고 commitAsync()를 매번 호출하면 됩니다.**

첨언을 하자면, 비동기 처리의 경우에는 주로 멀티쓰레드 환경을 이용해서 구현을 하게 되는데, 쓰레드는 주로 CPU 자원을 할당 받아 처리를 하기 때문에 CPU에 병목이 많이 걸리는 환경에서는 
비동기 처리가 커다란 오버헤드를 불러올 가능성이 존재합니다. 

따라서 비동기 처리의 경우는 Memory 자원보다는 CPU 자원을 신경써서 프로그래밍하는 습관을 들이도록합시다.

</div>
</details>

### 4. Blocking, Non-Blocking 의 차이점과, Sync 기반의 Non-Blocking 방식이 왜 성능 저하가 발생하는지 설명해주실래요? (고급)

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

Blocking 방식의 경우 해당 로직이 처음 실행부터 끝까지 제어권을 가지는 형태이고, Non-Blocking의 경우에는 로직의 처리 여부와 관계없이 나머지 로직이 계속 실행되는 방식을 의미합니다.

동기 기반의 논블로킹 로직의 경우 **해당 로직의 수행 과정에서 계속해서 polling 현상이 발생하기 때문에 Context Switching 현상이 지속적으로 일어납니다.** 
따라서 지속적인 Context Switching 현상때문에 delay가 많이 발생하게됩니다.

> 👉 **위에서 설명한 현상은 Spring Webflux와 Spring Data JPA를 같이 사용하는 경우에 발생하는 이슈입니다. 
> Spring Webflux의 경우 Event looping 기반으로 Non-Blocking 로직을 수행하고, JPA의 경우 내부적으로 transaction flush 시점에서 SQL을 한번에 처리하는 특징을 가집니다.
>  이 때 JPA는 SQL의 처리를 위해서 JDBC를 이용하게 되는데요, JDBC는 동기 처리를 기반으로 동작합니다. 따라서 위에서 서술한 이유로 인해서 실제 성능을 측정해보면 Spring MVC + JPA보다 Spring Webflux + JPA가 낮은 성능을 보입니다.**

</div>
</details>

### 5. Multi-Thread Programming에 대해서 설명해보실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

멀티쓰레드 프로그래밍은 **하나의 process 내부에서 여러개의 thread를 할당해서 logic을 수행하는 프로그래밍 기법입니다.**

* 장점
1. 멀티 프로세스 기반의 프로그래밍보다 효율적입니다. multi process를 기반으로 로직을 처리하면 위에서 서술한 context switching 비용 문제라던지,
 혹은 고아프로세스의 발생 위험 때문에 비효율적인 프로그래밍이 될수밖에 없습니다. 
(실제로도 C로 소켓 프로그래밍을 해보신 분이라면 아시겠지만, 멀티 프로세스로 프로그램을 작성하면 고아프로세스의 생성을 막기 위해서 블로킹 로직을 작성해야합니다. 이는 커다란 오버헤드를 불러일으킵니다.)
2. Thread는 여러개의 다른 쓰레드와 Heap 영역을 공유하기 때문에 이를 이용해서 thread간에 통신이 가능합니다.

* 단점
1. Heap 영역을 공유하기 때문에 Heap 영역의 특정 자원에 대해서 여러 개의 쓰레드가 동시에 접근하게 되면 Race condition에 빠질 위험이 있습니다.
2. 따라서 Race condition의 해소를 위해서 임계영역에 과도한 lock을 걸게되면 delay가 많이 발생하여 성능 저하를 일으킬 가능성이 존재합니다.

</div>
</details>

### 6. Process의 동기화에 대해서 설명해보실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

Process의 동기화란 **일정 자원에 대해서 한 번에 하나의 프로세스만이 접근하게 하는 것**을 의미합니다.

Process의 동기화가 보장되지 않는다면 Context Switching 이라거나, 혹은 Race condition 이슈때문에 해당 자원에 대해서 일관성이 보장되지 못하는 문제점들이 발생합니다.

1. Race Condition (경합, 경쟁상태): 여러 프로세스, 혹은 쓰레드가 동기화 메커니즘을 가지지 않고 공유 리소스에 접근하려는 현상
2. Critical Section (임계영역): 여러 thread가 동시에 접근해서는 안되는 code block을 의미한다

</div>
</details>

### 7. Critical Section에서 발생하는 동시 접근 문제들을 해결하기 위한 조건들을 말씀해주실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

1. 상호배제 (Mutual Exclusion): 하나의 프로세스가 Critical Section을 점유하고 있다면, 다른 프로세스가 Critical Section에 접근하는 것을 불허한다
2. 진행 (Process): Critical Section을 점유하는 프로세스, 혹은 쓰레드가 존재하지 않는다면 Critical Section으로 접근하려는 프로세스를 적절하게 선택하여 진입시킨다.
3. 유한대기 (Bounded Waiting): Critical Section으로 접근하려는 프로세스가 존재한다면, 다른 프로세스의 Critical Section에 대한 진입 요청은 유한한 횟수로 제한되어야한다. 
**(Critical Section의 접근에 대해서 프로세스의 기아현상을 방지하기 위해서이다.)**

</div>
</details>

### 8. Deadlock 현상이 무엇이고, 해결 방법을 설명해보실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

운영체제, 혹은 개발자가 작성한 소프트웨어가 잘못된 자원 관리 정책으로 인해서 둘 이상의 Process가 서로 점유한 자원을 반납하지 않고 선점하다가 뻗어버리는 현상을 의미한다.

Deadlock의 발생 조건은 아래와 같으며, 아래의 조건을 모두 만족시 Deadlock이 발생합니다.

1. 상호 배제 (Mutual Exclusion)
2. 점유 상태로 대기 (Hold and Wait)
3. 선점 불가 (다른 프로세스의 자원을 뺏어올 수 없는 경우)
4. 순환성 대기 (대기열에 Cycle이 발생하여 자신의 실행을 위해 자신의 자원 반납을 기다려야하는 상황)

현대의 프로그래밍 환경에서는 현실적으로 상호 배제 조건을 해소하기는 어렵습니다. 공유자원에 대해서 데이터 일관성은 보장되어야하는 사안이기 때문이죠.

따라서 Deadlock의 해소 방안은, 일단 Deadlock이 발생하지 않도록 프로그래밍하는 것이 제일 중요하고, 그렇지 않다면 OS 차원에서 
Deadlock이 발생한 Process를 회복시키거나, 혹은 Deadlock을 회복시키지 못한다면 무시하는 것이 방법이 되겠습니다.

</div>
</details>

### 9. Process의 Starvation(기아) 현상에 대해서 설명해보실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

여러개의 Process에 대해서 동시성을 보장하기 위해서 CPU 스케쥴링이 동작합니다. 이 과정에서 특정 프로세스가 낮은 우선순위를 가지거나, 혹은 비선점 방식으로 인해서 
이전의 Process가 CPU를 꽉 잡고 자신의 자원을 내려놓지 않는 경우 이후의 프로세스가 실행되지 않는 현상이 발생합니다. 이러한 경우를 **Process Starvation**이라고 부릅니다.

해소 방안은, 기아현상이 우려되는 Process에 대해서 임의로 우선순위를 높여주거나, 혹은 CPU 스케쥴링을 비선점 방식으로 진행시켜서 우선순위가 낮은 프로세스가 실행되는 것을 보장하는 방법이 있습니다.

물론 Round Robin 같은 비선점 스케쥴링 방식은 모든 프로세스가 실행되는 것이 보장되는 스케쥴링 방식이지만, 특정 프로세스의 burst time이 긴 경우 해당 프로세스의 완료에 대해 delay가 발생할 우려가 있습니다.

따라서 스케쥴링에 대해서는 trade-off를 잘 따져가면서 프로세스의 기아현상을 해소시키는 것이 바람직하겠습니다.

**(소위 말하는, 프로그래밍 세상에서는 은탄환이 없다고 비유들을 많이 하십니다.)**

</div>
</details>

### 10. Semaphore와 Mutex 방식에 대한 설명과, 차이점을 설명해주실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

Mutex의 경우 Critical Section에 대해서 한 번에 하나의 프로세스만이 접근하도록 허용하는 방식이고, Semaphore은 유한의 개수의 프로세스가 한번에 Critical Section에 진입하는 것을 허용하는 방식입니다.

위의 특징 때문에 Semaphore은 Mutex가 될수 있고, Mutex는 Semaphore가 될 수 없다는 특징을 가집니다.

거기에 더해서, Mutex의 경우 Mutex 자체가 자원을 가지고 가진 자원에 대해서 책임을 지는 반면에, Semaphore은 자원을 가질 수 없다는 특징도 있습니다.

그리고 Mutex의 경우 Life Cycle이 해당 Process에 제한된다는 특징이 있고, Semaphore은 OS 상에서 파일로 존재하기 때문에 Mutex에 비해서 생애주기가 길다는 특징도 있습니다.

마지막으로 설명드리자면, Mutex의 경우에는 Mutex를 점유하는 쓰레드가 스스로 lock을 해제시켜줘야만 다른 쓰레드가 Critical Section에 접근이 가능하다는 특징을 가지지만, Semaphore의 경우 lock을 가지지 않은 다른 스레드가 Semaphore를 해제하는게 가능하다는 특징도 있습니다.

</div>
</details>

### 11. Virtual Memory에 대해서 설명해보실래요?

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

가상 메모리의 경우 Memory와 Storage의 Swap 영역을 합친 개념입니다. 그리고 이 두개의 조합을 가상화 시켜서 OS는 Memory를 논리적으로 관리합니다. 이러한 특징 때문에 이 두개의 메모리 조합을 가상메모리라고 부르는 것입니다.

OS의 경우 가상메모리에 대한 물리적 위치를 따지지 않는다는 특징도 있습니다. 따라서 Memory 자원과 Swap Memory 자원을 구분하지 않습니다.

그러나, Swap Memory의 경우는 Storage에 기반한 Memory 영역이기 때문에 (물론, 일반 Storage에 비해서는 빠른 속도를 가집니다만, Memory에 비해서는 그래도 속도가 느립니다) Swap Memory를 사용시 process의 진행에 대해서 
처리 속도에 delay가 발생한다는 특징이 있습니다. 따라서 가급적이면 개발자는 swap memory를 사용하지 않도록 주의하여 프로그래밍을 할 필요가 있습니다.

</div>
</details>

### 12. 캐시의 지역성(locality)에 대해서 설명해보실래요? 

<details>
<summary>접기/숨기기</summary>
<div markdown="1">

memory 상에서는 자주 접근되는 데이터, 자주 접근되지 않는 데이터가 존재합니다. (Hot/Cold Data 라고 부르기도 합니다)

그리고 OS는 자주 접근되는 데이터에 대해서는 쓰로풋을 늘리기 위해 Memory의 상위 계층의 캐시에다가 해당 데이터를 임시 저장하고 사용하는 정책을 수행합니다.

이 때 **자주 사용되는 데이터**에 대한 선정 기준이 필요한데, 이 때 등장하는 개념이 **공간적 지역성**과 **시간적 지역성** 이라는 개념이 등장합니다.

위의 질문과 관계가 없지만, 추가적으로 설명하자면 OS는 잉여 메모리 공간을 할당하여 page cache라는 영역을 할당합니다. 그리고 OS는 자주 참조되는 파일의 내용을 page cache에 임시로 저장해놓았다가 다음에 read/open을 수행할 시 
빠른 속도를 지원할 수 있습니다. **이는 Apache Kafka가 사용하는 방식으로, Apache Kafka는 모든 log, 혹은 commit offset topic을 log 파일의 형태로 file system에 저장을 하기 때문에, 자주 참조되는 log(open된 log segment)에 대해서는 이를 
page cache에 담아두었다가 나중에 참조가 될 시 빠른 속도로 read/write를 할수있게 되는 것입니다.**

따라서 Apache Kafka는 Redis와 달리 Memory 기반이 아닌 F/S 기반의 메세지 브로커임에도 불구하고 빠른 처리 속도를 보일 수 있는 이유가 위에 설명한 내용에 있는 것입니다.

</div>
</details>