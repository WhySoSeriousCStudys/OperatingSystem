# 01. Interrupt

: 프로그램을 실행하는 도중에 예기치 않은 상황이 발생할 경우 현재 실행중인 작업을 즉시 중단하고, 발생된 상황에 대한 우선 처리가 필요함을 CPU에게 알리는 것.

지금 수행중인 일보다 더 중요한일(i/o interrupt, priority)이 발생하면 그 일을 먼저 처리하고 나서 하던 일을 계속 해야함.

하드웨어: 시스템 버스를 통해 CPU에 신호를 보내 인터럽트를 발생시킬 수 있음

- 외부 인터럽트: 입출력 장치, 타이밍 장치, 전원 등 외부적인 요인으로 발생
- 내부 인터럽트: Trap 이라고 부르며, 잘못된 명령이나 데이터를 사용할 때 발생 (Exception)

소프트웨어: system call을 실행하여 인터럽트 발생

### 인터럽트 발생 시 처리 과정

주 프로그램이 실행되다가 인터럽트 발생

현재 수행중인 프로그램을 멈추고, 상태 레지스터와 PC등을 스택에 잠시 저장한 뒤에 인터럽트 서비스 루틴으로 간다

![image](https://user-images.githubusercontent.com/46514182/184569270-b8f96957-5253-4a1b-a166-87081e5b9b01.png)  

> CPU가 인터럽트 되면, CPU는 하던일을 중단하고, 즉시 고정된 위치로 실행을 옮긴다.
> 

> 이러한 고정된 위치는 일반적으로 인터럽트를 위한 서비스 루틴이 위치한 시작 주소를 가지고 있다.(interrupt vector)
> 

> 이후, 인터럽트 서비스 루틴을 실행한다
> 

> 인터럽트 서비스 루틴의 실행이 완료되면, CPU는 인터럽트 되었던 연산을 재개한다.
> 

`interrupt → context switch → device controller의 레지스터에 정보 로드`

만약 인터럽트 기능이 없었다면, 컨트롤러는 특정한 어떤 일을 할 시기를 알기 위해 계속 체크를 해야함

→ 이를 **폴링(Polling)**이라고 함

폴링을 하는 시간에는 원래 하던 일에 집중할 수가 없게 되어 많은 기능을 제대로 수행하지 못하는 단점이 있음

즉, 컨트롤러가 입력을 받아들이는 방법(우선순위 판별방법)에는 2가지가 있음

> 폴링 Polling
> 

컨트롤러와 호스트 사이에 생산자 소비자 관계를 조장하기 위해 두 개의 비트가 사용됨. 제어기는 상태 레지스터의 비지(busy) 비트를 통해 자신의 상태를 나타냄.

(비트 설정: 비트에 1을 쓴다 = 바쁘다, 비트 소거: 비트에 0을 쓴다 = 안바쁘다, 다음 명령을 받아들일 준비 완료)

**호스트가 반복적으로 비지 비트가 소거 되었는지를 계속 검사하는 것을 폴링이라고 함!**

근데 이렇게 계속 검사하는 기간이 짧으면 괜찮은데 이게 길어지면 호스트는 다른 태스크로 전환해서 다른 일을 하다 오는게 효율적임. 하지만 컨트롤러가 일을 끝냈는지 알 수가 없음. 따라서 어떤 장치들은 호스트가 빨리 서비스해주지 않으면 자료를 잃어버리게 됨

폴링 연산은 효율적이지만 호스트가 계속 폴링 연산을 하는데 장치가 서비스 준비가 완료되는데 시간이 오래걸린다면, 폴링은 비효율적이 되고, 다른 유용한 CPU 처리가 계속 지연됨. 이와 같은 경우에는 하드웨어 컨트롤러가 자신의 상태가 바뀔 때 CPU에 그것을 통보해주는 것이 반복적으로 계속 폴링을 하는 것보다 효율적임

→ 입출력 장치가 CPU에게 자신의 상태 변화를 통보하는 하드웨어 기법을 **인터럽트(interrupt)**라고 함.

![image](https://user-images.githubusercontent.com/46514182/184569237-eba1a7c3-e351-4d1e-9a87-8aeee36be82b.png)  

인터럽트 방식은 하드웨어로 지원을 받아야 하는 제약이 있지만, 폴링에 비해 신속하게 대응하는 것이 가능하다. 따라서 **'실시간 대응'**이 필요할 때는 필수적인 기능이다.

즉, 인터럽트는 **발생시기를 예측하기 힘든 경우에 컨트롤러가 가장 빠르게 대응할 수 있는 방법**
이다.

> **나올 수 있는 질문**
> 
> 
> 인터럽트 발생 및 처리과정을 설명하세요.
> 인터럽트 기능이 없다면 사용하는 방식에 대해 설명하세요.
> Context Switching이란 무엇인가요?
>
