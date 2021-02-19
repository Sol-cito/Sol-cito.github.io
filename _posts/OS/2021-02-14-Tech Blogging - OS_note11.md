---
layout: post
title: "[OS] 쓰레드의 이해 Part 2"
subtitle: "Synchronization, semaphore, deadlock and starvation"
background: '/img/bg_technology.jpg'
categories: technology-os
---

#### 쓰레드와 동기화(Synchronization) 이슈

쓰레드는 한 프로세스의 자원을 공유한다고 하였다.

그러다보니 여러 쓰레드가 공유하고있는 한 프로세스의 변수 등의 Data에 쓰레드가 동시에 접근하게 된다면, 

**의도치 않은 비정상적인 결과**가 도출될 가능성이 존재한다.

따라서 이러한 상황을 막기 위해 **동기화이슈를 관리**해야 하는 상황이 발생한다.

<br>

아래 python 예시 코드를 보자.

```python
import threading # 파이썬의 쓰레드 생성 외부 라이브러리

g_count = 0 # 글로벌 변수 선언

def thread_main(): # 글로벌 변수 g_count에 1을 십만 번씩 더하는 함수
    global g_count
    for i in range(100000):
        g_count +=1
        
threads = []

for i in range(50): # list에 50개의 쓰레드를 붙인다.
    th = threading.Thread(Target = thread_main)
    threads.append(th)

for th in threads: # list 내 쓰레드의 함수를 실행시킨다.
    th.start()

# join() 은 쓰레드 간 동기화 함수
# 모든 쓰레드의 실행이 종료될 때까지 대기하게 만듬.
for th in threads: 
    th.join()
    
print('g_count = ', g_count)
```

위 코드를 실행시켰을 때, **동기화 이슈가 발생할 수 있다.**

그 이유는 아래와 같다.

- g_count +=1 이라는 코드를 실행시켰을 때, 컴퓨터 내부 구조는 다음과 같이 동작한다(컴퓨터 구조에 대한 이해 필요).
  - (1) g_count의 현재값을 register에 저장한다.
  - (2) register 값을 1 증가시킨다.
  - (3) register에 기록된 값을 g_count 메모리 값에 덮어씌운다.

- 그런데 현재 위의 thread_main 함수에서 위 (1), (2), (3)을 실행하는 반복문이 100000번 이므로, **만약 연산 시간이 길다면 쓰레드 사이에서 반복문 중간에 Context Switching이 발생할 수 있다** (반복문이 100000번이 아니라 100번 정도라면, 연산 속도가 짧으므로 Context Switching이 발생하지 않음).

- Context Switching이 발생할 경우 다음 쓰레드가 thread_main 함수를 실행시키면서, **동일 register를 초기값 0으로 만들고 g_count 값을 덮어씌우고 연산을 시작한다.**
- 다시 Context Switching이 발생하면, **동일한 register에 값이 덧씌워지고 다시 끝난 지점부터 연산을 시작하게 된다.**

- 이처럼 쓰레드 사이의 Context Switching이 빈번하게 발생하다 보면 register에 저장된 값이(변수값) **계속 변화하므로 연산 결과값이 제대로 반영되지 못할 수 있다.**

이것이 바로 **쓰레드 간 변수를 공유함에 따라 발생하는 쓰레드 동기화 이슈**의 한 예이다.

<br>

그럼, 저기서 문제가 되는 thread_main() 함수의 연산 부분을 아래와 같이 바꾸면..

```python
def thread_main():
    global g_count
    lock.acquire() # 락을 걸어준다. - 파이썬 제공 API
    for i in range(100000): # 해당 for문은 임계영역
        g_count +=1
    lock.release() # 락이 풀린다.
    
lock = threading.Lock()
```

lock.acquire() 함수를 특정 쓰레드가 실행하게 되면, release될 때 까지는 다른 쓰레드가 acquire() 함수를 실행하지 못하고 기다리게 된다.

따라서 for문 연산은 **하나의 쓰레드만 접근할 수 있게 된다(동시접근을 막음).**

위와 같은 기법을 **Mutual Exclusion**(상호 배제)라고 하며, 

쓰레드끼리 겹치는 for문 연산과 같은 영역을 **임계 영역(critical section)**이라고 하고,

g_count와 같은 겹치는 자원을 **임계 자원(critical resource)**이라고 한다.

이 때, 임계 영역에 여러 스레드가 들어갈 수 있게 만드는 기법을 **세마포어**라 한다.

<br>

이처럼, 임계구역에 대한 접근을 막기 위해 Locking 메커니즘이 필요하다.

Locking 메커니즘은 크게 2가지 방식이 있는데,

- **Mutex(Binary semaphore)** : 임계구역에 하나의 스레드만 들어갈 수 있음.
- **Semaphore** : 임계구역에 여러 스레드가 들어갈 수 있음. Counter를 두어서 동시에 접근할 수 있는 허용 가능한 스레드 수를 제어함.

<br>

#### 세마포어(Semaphore)

세마포어는 counter를 두어 자원 접근의 스레드 개수를 제어하는 방법이다.

논리 자체는 단순한데,

어떤 프로세스가 **임계영역에 진입할 때 couter 변수의 값을 -1,**

**임계영역에서 나올 때 counter 변수의 값을 +1**로 하여,

**counter 변수의 값이 특정 수 이상일 경우에만 임계영역에 접근을 허용**하는 방식이다.

수도 코드로 표현하면 아래와 같다.

```c
// S는 세마포어 카운터를 의미
P(S) : wait(S){ // 임계영역 진입시의 수도코드 (진입 가능한지 검사함)
				while S <= 0 // S가 0 이하이면 loop안에서 기다린다(wait)
				;
		S--; // 임계영역에 접근하면 S를 -1 하여 다른 프로세스의 접근을 막는다.
}

V(S) : signal(S){ // 임계영역에서 빠져나올 때의 수도코드
		S++; // 임계영역에서 나오면 S를 +1 하여 다른 프로세스의 접근을 허용한다.
}
```

자 그런데, 위 wait 수도코드의 경우 '기다림'을 표현하기 위해 **while loop**문을 썼다.

왜냐하면, 기본적으로 컴퓨터 프로그래밍에서는 **CPU의 중단을 표현하는 것은 불가능하고, 끊임없이 코드를 실행해야 하기 때문이다**.

이러한 loop문을 이용하는 것을 **바쁜 대기(busy waiting)**라 하는데, 

이 방법은 당연하게도 CPU에 부하를 걸리게 한다(쓸데없는 무한반복문).

이러한 부하를 줄이는 방법은 **운영체제 기술 - 대기큐**를 사용하는 방법이다.

```c
P(S) : wait(S){
    	S->count--;
    	// S가 0 이하인 경우 queue에 넣고 sleep시킨다.
		if (S <= 0){
			add this process to S->queue;
			block();
		}
}

V(S) : signal(S){
    	S->count++;
    	// S가 양수인 경우 queue에 넣었던 프로세스를 깨운다.
		if (S >= 1){
            remove a process P from queue;
            wakeup();
        }
}
```

리눅스에서는 **POSIX세마포어**라는 기본적인 세마포어 API를 정의해놓았다.

<br>

#### 교착상태(Deadlock)

'무한 대기상태'로, 두 개 이상의 작업이 서로 상대방의 작업이 끝나기만을 기다리고 있는 상태를 의미한다.

![image](https://media.geeksforgeeks.org/wp-content/cdn-uploads/gq/2015/06/deadlock.png)

(출처 : https://www.geeksforgeeks.org/introduction-of-deadlock-in-operating-system/)

<br>

위 그림처럼 Process 1은 Resource 2을, Process 2는 Resource 1을 점유하려고 대기(wait)하고 있으나 

Process 1, Process 2 각각이 상대방이 원하는 자원을 점유하고 있기 때문에 더이상 프로그램의 실행이 되지 않는다.

**프로세스, 쓰레드 둘 다 이와 같은 상태가 발생할 수 있다.**

교착상태는 **프로세스를 하나씩 실행하는 배치처리 시스템에서는 발생하지 않는 현상**이다.

교착상태가 발생하는 조건은 아래 4가지 조건이 있다.

- 상호배제(Mutual exclution) : 프로세스들이 필요 자원에 대한 배타적 통제권을 요구함.
- 점유대기(Hold and wait) : 프로세스가 할당된 자원을 가지고 있으면서 동시에 다른 자원을 요구, 대기함.
- 비선점(No preemption) : 한 프로세스가 어떤 자원의 사용을 끝낼 때 까지 그 자원을 뺏을 수 없다.
- 순환대기(CIrcular wait) : 각 프로세스는 순환적으로 다음 프로세스가 요구하는 자원을 가지고 있다.

교착상태의 해결방법은 위 4가지 상태 중 최소 하나의 조건이 발생하지 않도록 하는 것인데,

**현실적으로 실제 상황에서 위 4가지 중 어떤 조건이 발생할 지를 알 수 없으므로,**

**문제 발생 시점에 코드로 해결하는 것이 가장 현실적이다.**

물론 문제의 예방 및 회피는 가능하며, 그 방법은 매우 다양하다.

<br>

#### 기아상태(Starvation)

특정 프로세스의 **우선순위**가 낮아서, 원하는 자원을 계속 할당받지 못하는 상태를 뜻한다.

대표적인 예로 **다익스트라의 식사하는 철학자 문제**가 있다.

> 5인의 철학자가 원형 테이블에서 포크 2개로 식사를 하고 싶어하며, 테이블에는 포크가 5개 놓여있다. 철학자가 동시에 자신의 왼쪽 포크를 집으면, 그 누구도 오른손에 포크를 쥘 수 없으므로 아무도 식사를 할 수 없어 기아가 된다(...)

기아상태의 해결은 **우선순위의 변경**이며, 그 메커니즘에도 여러가지 방법이 있으나 딱 정해진 방법은 존재하지 않는다.

프로그램의 코드 레벨에서 해당 상태를 수정해냐가야 한다.

<br>

두 현상의 차이점 : 

- 데드락 : 여러 프로세스가 **동일 자원** 점유를 요청할 때 발생함.

- 기아 상태 : 여러 프로세스가 **부족한**자원을 점유하기 위해 경쟁할 때 발생함.

<br>
---
위 내용은 '패스트캠퍼스'의 컴퓨터공학 강좌 내용을 요약 정리한 것임을 밝힙니다.
(https://www.fastcampus.co.kr/)