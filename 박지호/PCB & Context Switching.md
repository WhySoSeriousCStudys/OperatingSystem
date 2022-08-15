# 03. PCB & Context Switching

### Process Management

: CPU가 프로세스가 여러개일 때, CPU 스케줄링을 통해 관리하는 것을 말함

이때, CPU는 각 프로세스들이 누군지 알아야 관리가 가능함.

프로세스들의 특징을 갖고있는 것: `process metadata`

<aside>
💡 **Process Metadata**
- Process ID
- Process State
- Process Priority
- CPU Registers
- Owner
- CPU Usage
- Memory Usage

</aside>

이 메타데이터는 프로세스가 생성되면 `PCB(Process Control Block)` 이라는 곳에 저장됨

### Process Control Block (프로세스 제어 블록, 태스크 제어 블록)

: 프로세스 메타데이터들을 저장해 놓는 곳. 특정 프로세스와 연관된 여러 정보를 수록하며 다음과 같은 정보를 포함함

![Untitled](03%20PCB%20&%20Context%20Switching%205343168be9944fb99fe0adb134b34ecf/Untitled.png)

![Untitled](03%20PCB%20&%20Context%20Switching%205343168be9944fb99fe0adb134b34ecf/Untitled%201.png)

<aside>
💡 - **프로세스 상태**: 상태는 new(새로운), ready(준비완료), running(실행), waiting(대기), halted(정지)
- **프로세스 카운터**: 프로그램 카운터는 이 프로세스가 다음에 실행할 명령어의 주소를 가리킨다
- **CPU 레지스터들**: CPU 레지스터는 컴퓨터 구조에 따라 다양한 수와 타입을 가진다. 레지스터에는 여러가지 상태정보들과 프로그램카운터는 나중에 프로세스가 계속 올바르게 실행되도록하기 위해서, 인터럽트 발생 시 저장되어야 함
- **CPU 스케줄링 정보**: 이 정보는 프로세스의 우선순위, 스케줄 큐에 대한 포인터와 다른 스케줄 매개변수들을 포함함
- **메모리 관리 정보**: 운영체제에 의해 사용되는 메모리 시스템에 따라 기준 레지스터와 한계 레지스터 값, 운영체제가 사용하는 메모리 시스템에 따라 페이지 테이블 또는 세그먼트 테이블 등과 같은 정보를 포함함
- **회계 정보**: CPU 사용 시간과 경과된 실시간, 시간 제한, 계정 번호, job 또느 process 번호 등 포함
- **입출력 상태 정보**: 프로세스에게 할당된 입출력 장치들과 열린 파일의 목록 등을 포함함

</aside>

→ 프로세스 제어 블록은 프로세스마다 달라지는 모든 정보를 저장하는 저장소의 역할을 함

```c
프로그램 실행 → 프로세스 생성 → 프로세스 주소 공간에 (코드, 데이터, 스택) 생성
→ 이 프로세스의 메타데이터들이 PCB에 저장
```

### PCB 가 필요한 이유?

CPU에서 프로세스의 교체 작업(interrupt 발생 등)이 이루어지는데 앞으로 다시 수행할 대기중인 프로세스에 관한 정보들은 PCB에 저장해 두고 나중에 다시 해당 프로세스를 사용할 때 PCB에 있는 정보를 올려서 이어서 실행하기 위함

### PCB 관리?

Linked List 방식으로 관리함

PCB List Head에 PCB들이 생성될 때마다 붙게 된다. 주소값으로 연결이 이루어져 있는 연결 리스트이기 때문에 삽입 삭제가 용이함.

→ 프로세스가 생성되면 해당 PCB가 생성되고, 프로세스가 완료되면 제거됨

### Context Switching

: CPU가 다른 프로세스로 교환하려면, 이전 프로세스의 상태를 PCB에 보관하고 새로운 프로세스의 상태를 PCB에 읽어서 레지스터에 적재하여 보관된 상태를 복구하는 작업

보통 인터럽트가 발생하거나, 실행 중인 CPU 사용 허가 시간을 모두 소모하거나, 입출력을 위해 대기해야하는 경우에 Context Switching이 발생함

`즉, 프로세스가 Ready → Running, Running → Ready, Running → Waiting처럼 상태 변경 시 발생함`

**Context Switching의 Overhead?**

: context switching이 진행될 동안 시스템은 아무런 유용한 일을 하지 못하기 때문에 context switching을 하는 동안은 overhead가 발생함.

감수해야하는 overhead!

> 프로세스를 수행하다가 입출력 이벤트가 발생해서 대기 상태로 전환시킴
이때, CPU를 그냥 놀게 놔두는 것보다 다른 프로세스를 수행시키는 것이 효율적
>