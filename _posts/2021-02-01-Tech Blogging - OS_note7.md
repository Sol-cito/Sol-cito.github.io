---
layout: post
title: "[OS] 프로세스와 스케쥴러의 이해 Part 3"
subtitle: "Interrupt에 대하여"
background: '/img/bg_technology.jpg'
categories: technology
---


#### 인터럽트란?

CPU가 프로그램을 실행하고 있을 때, 

**입출력 하드웨어 등의 장치나 또는 예외상황이 발생하여 처리가 필요할 경우**에 CPU에 이벤트를 알려서 처리하는 기술이다.

<br/>

앞선 Part 2 에서 살펴봤던 것 처럼, 어떤 프로세스가 외부자원을 활용해야 할 때, 

그 프로세스는 **Wait 상태**로 진입한다.

그리고 CPU는 다른 프로세스를 처리하게 되는데,

이 프로세스가 외부자원 이용이 다 끝나고 Wait상태에서 돌아오기 위해서는 **CPU에 그 사실을 알려야만 하고,**

**운영체제 안의 Kernel에서 특정 코드 처리를 통하여** CPU에게 이벤트를 알리는 것이 바로 **인터럽트**이다.

<br/>

인터럽트가 필요한 경우는.

- 프로세스 Running 도중, 현재 프로세스를 중단시키고 다른 프로세스로 교체해야 할 때 ->**선점형 스케쥴러**에서 반드시 필요함.
- I/O device와의 커뮤니케이션 - 외부자원 사용 완료 시, 프로세스를 wait상태에서 ready상태로 만들어줘야 함.
- 예외상황 핸들링 : CPU가 프로그램을 실행중, I/O 장치에서 문제가 발생할 경우, 또는 예외상황이 발생할 경우,**CPU가 해당 처리를 할 수 있도록 CPU에 알려줘야 함.**
  - 메모리나 하드디스크 등 하드웨어 장치가 발열이 심해져서 작동이 어려울 경우
  - 프로그램 실행 중 **Runtime Error**가 발생할 경우(ex) 0으로 나누기) : 프로그램 running이 더이상 불가능하므로, Process를 Kill 해줘야 한다 --> 인터럽트가 필요함

<br/>

##### 인터럽트 발생 종류 1 - Runtime Error

0으로 나눌 때, Overflow가 발생할 때, NullPointerException 등...

```c
// Divide-by-zero 에러
int main(){
    int data = 1;
    int divider = 0;
    data = data / divider; // runtime error 발생
    return 0;
}
```

위 코드에서 Runtime error는 **컴파일 과정이 아니라, CPU가 해당 프로그램을 실행하는 동안 발생한다.**

Runtime Error가 발생하면 **운영체제는 해당 Process를 Kill하고, 특정 메시지를 출력한다 (Exception 메시지)**

---> 위와 같은 0으로 나누는 상황에 대한 **Devide-by-zero 예외 루틴**을 실행하는 것이다.

이는 프로그램 내부에서 발생했기 때문에 **내부 인터럽트(또는 Software Interrupt)**로 분류한다.

<br/>

##### 인터럽트 발생 종류 2 - Timer Interrupt

선점형 스케쥴러는 특정 시간이 지나면 Process를 강제로 교체하게 되는데,

이 때 필요한 것이 **Timer Interrupt**이다. 특정 시간이 지날 때 마다 저절로 인터럽트가 걸리고, Process가 교체된다.

타이머 인터럽트는 프로그램 외부에서 발생했기 때문에 **외부 인터럽트(또는 Hardward Interrupt)**로 분류한다.

<br/>

##### 인터럽트 발생 종류 3 - I/O interrupt

프린터, 마우스, 키보드 등 하드웨어에서 입출력이 끝날 때 인터럽트가 걸린다.

이 또한 프로그램 외부에서 발생했기 때문에 **외부 인터럽트**로 분류한다.

<br/>

#### 시스템 콜 인터럽트

>시스템 콜도 Interrupt다!

당연하게도, 시스템콜도 인터럽트이다. 

시스템콜이 실행되면 운영체제는 다음과 같은 절차를 밟게 된다.

- eax 레지스터에 시스템 콜 번호를 넣고, ebx레지스터에는 시스템 콜 parameter를 넣는다.
- Software Interrupt 명령을 호출하면서 0x80값 (Software Interrupt 명령의 값 - op code)을 넘겨준다.
- 0x80 값을 받은 CPU는 **사용자모드를 커널모드로 변환**한 후, **IDT(Interrupt Descriptor Table)**에서 0x80에 해당하는 주소(함수)를 찾아서 실행한다. -- > 이 함수는 system_call() 함수가 될 것이다.
- 이 system_call() 함수는 종류가 여러 개가 있다.
  - 1번 : sys_exit() / 2번 : sys_fork() / 3번 : sys_read .....etc
- 그 중에서 eax레지스터에 넣어놓았던 시스템 콜 번호와 매칭되는 함수를 호출한다.
- 함수 호출 시, parameter로 ebx레지스터에 저장해놓았던 parameter를 넘긴다.
- 커널모드에서 함수 실행이 종료되면 **다시 커널 모드에서 사용자모드로 변경**하고, 다시 해당 프로세스의 다음 코드를 진행한다.

![image](https://media.geeksforgeeks.org/wp-content/uploads/dual_mode.jpeg)

(출처 : https://www.geeksforgeeks.org/dual-mode-operations-os/ )

따라서, 프로세스를 실행할 때 **운영체제는 사용자모드와 커널모드를 계속 번갈아가면서 실행함을 알 수 있다.**

<br/>

#### 인터럽트와 IDT(Interrupt Descriptor Table)

인터럽트는 미리 정의되어 각각 번호와 실행코드를 가리키는 주소가 IDT에 기록되어 있다.

이 IDT는 **컴퓨터 부팅 시 운영체제 내부에 운영체젠가 기록한다.**

따라서 인터럽트가 발생하면, op code에 맞는 주소(시스템콜의 경우 0x80)를 찾아가서 해당 시스템콜을 확인 후 실행한다.

- 리눅스의 IDT 예시 : 
  - 0~31번 : 내부 인터럽트 예외상황 (일부는 정의 안된채로 남겨져 있음)
  - 32~47번 : 하드웨어 인터럽트 (주변장치 종류 / 갯수에 따라 변경 가능)
  - 128 : 시스템 콜 (= 0x80을 십진수로 표현하면 128)

<br/>

인터럽트와 프로세스를 다시 정리하자면...

- 프로세스 실행 중 Interrupt 발생(시스템콜 or 예외상황 등)
- 프로세스는 실행을 중단하고 **IDT로 가서 인터럽트 번호에 해당하는 함수를 kernel모드에서 실행**
- 운영체제가 인터럽트 처리 함수 실행
- 다시 사용자모드로 돌아가 프로세스를 실행

---
위 내용은 '패스트캠퍼스'의 컴퓨터공학 강좌 내용을 요약 정리한 것임을 밝힙니다.
(https://www.fastcampus.co.kr/)
