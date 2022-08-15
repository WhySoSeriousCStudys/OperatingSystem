# 02. System Call

Process 생성과 제어를 위한 함수 호출

> OS의 2가지 모드
- **kernel mode**: 모든 시스템 메모리 접근 가능. 모든 cpu 명령 실행 가능
- **user mode**: 사용자 애플리케이션 실행. 하드웨어 직접 접근 불가. system call 호출 시 일시적으로 kernel mode 로 전환
> 

→ 운영 체제의 커널이 제공하는 서비스에 대해, 응용 프로그램의 요청에 따라 커널에 접근하기 위한 인터페이스

→ 커널 영역의 기능을 사용자 모드가 접근하게 도와주는 기능을 system call이라고 함

→ 보통 고급 프로그래밍 언어들로 작성된 프로그램들은 직접 system call을 사용할 수 없기 때문에 고급 API(운영체제가 제공하는)를 통해 system call에 접근하게 하는 방법

![Untitled](02%20System%20Call%20295f6576784141dbb6721223f07410e6/Untitled.png)

system call을 사용하는 이유: 사용자 어플리케이션이 운영체제의 치명적인 데이터의 수정/삭제를 막기 위함

> **System Call 의 유형**
> 
1. 프로세스 제어 (process & job manipulation or control)
    
    → 프로세스 생성, 종료와 같이 프로세스를 처리함
    
2. 파일 조작 (file manipulation)
    
    → 파일 생성, 파일 읽기, 파일 쓰기와 같은 파일 조작 담당
    
3. 장치 조작 (device manipulation)
    
    → 장치 버퍼에서 읽기, 쓰기와 같은 장치 조작 담당
    
4. 정보 유지 보수 (information maintenance)
    
    → 운영체제와 사용자 프로그램 간의 정보 전송을 처리
    
5. 통신 (communications)
    
    → 프로세스 간 통신 관리, 통신 연결을 만들고 삭제하는 작업을 관리
    

- fork(), exec(): 새로운 proecss 생성하는 시스템 호출
- wait(): process(parent)가 만든 다른 process(child)가 끝날 때 까지 기다리는 명령어

> **fork()**
: 현재 실행중인 프로세스와 동일한 기능을 하는 새로운 process를 생성할 때 사용. → copy
> 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    printf("pid : %d", (int) getpid()); // pid : 29146
    
    int rc = fork();					// 주목
    
    if (rc < 0) {
        exit(1);
    }									// (1) fork 실패
    else if (rc == 0) {					// (2) child 인 경우 (fork 값이 0)
        printf("child (pid : %d)", (int) getpid());
    }
    else {								// (3) parent case
        printf("parent of %d (pid : %d)", rc, (int)getpid());
    }
}
```

**[ 반환값 ]**

+ 값: fork()에서 1 이상을 return 받는 프로세스는 parent 프로세스이며, return 값은 자식 프로세스 id

0: fork()에서 0을 return 받는 프로세스는 child 프로세스 자신임

-1: 오류가 발생하여 child 프로세스가 생성되지 않았다는 뜻이다. 

parent랑 child의 순서는 non-deterministic함. 즉, 순서를 확신할 수가 없음.

스케줄러가 결정하는 일 이기 때문에, parent가 먼저 실행되기도 하고, child가 먼저 실행되기도 함

운영체제는 위의 똑같은 2개의 프로그램이 동작한다고 생각하고 fork()를 return 시킴. → child process는 main에서 시작하지 않고, if 문부터 시작하게 됨.

> **wait()**
: child 프로세스가 종료될 때까지 기다리는 작업
> 

위의 예시에 `int wc = wait(NULL)` 만 추가함.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
    printf("pid : %d", (int) getpid()); // pid : 29146
    
    int rc = fork();					// 주목
    
    if (rc < 0) {
        exit(1);
    }									// (1) fork 실패
    else if (rc == 0) {					// (2) child 인 경우 (fork 값이 0)
        printf("child (pid : %d)", (int) getpid());
    }
    else {								// (3) parent case
        int wc = wait(NULL)				// 추가된 부분
        printf("parent of %d (wc : %d / pid : %d)", wc, rc, (int)getpid());
    }
}
```

프로세스 종료를 기다린다. 주로 fork()를 이용해서 child 프로세스를 생성했을 때 사용함. 

wait()를 쓰면 child 프로세스가 종료할 때 까지 해당 영역에서 부모 프로세스가 sleep() 모드로 기다리게 됨. → 이는 child 프로세스와 parent 프로세스의 동기화를 위한 목적으로, parent 프로세스가 child 프로세스보다 먼저 종료되어서 child process가 고아 프로세스가 되는 것을 방지하기 위한 목적.

parent가 먼저 실행되더라도, wait()는 child가 끝나기 전에는 return하지 않으므로, 반드시 child가 먼저 실행됨.

```c
// 실행결과
pid : 29146
child (pid : 29147)
parent of 29147 (wc : 29147 / pid : 29146)
```

> **exec()**
: 단순 fork()는 동일한 프로세스의 내용을 여러번 동작할 때 사용. child 에서는 parent와 다른 동작을 하고 싶을 때 exec()를 사용할 수 있음.
> 

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
    printf("pid : %d", (int) getpid()); // pid : 29146
    
    int rc = fork();					// 주목
    
    if (rc < 0) {
        exit(1);
    }									// (1) fork 실패
    else if (rc == 0) {					// (2) child 인 경우 (fork 값이 0)
        printf("child (pid : %d)", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("wc");		// 내가 실행할 파일 이름
        myargs[1] = strdup("p3.c");		// 실행할 파일에 넘겨줄 argument
        myargs[2] = NULL;				// end of array
        execvp(myarges[0], myargs);		// wc 파일 실행.
        printf("this shouldn't print out") // 실행되지 않음.
    }
    else {								// (3) parent case
        int wc = wait(NULL)				// 추가된 부분
        printf("parent of %d (wc : %d / pid : %d)", wc, rc, (int)getpid());
    }
}
```

exec가 실행되면, execvp( **실행 파일**, 전달 인자 ) 함수는, code segment 영역에 parameter로 전달받은 **실행 파일**의 코드를 읽어와서 현재 parent 프로세스 공간의 exec 인자에 있는 실행파일에 대한 TEXT, DATA, BSS영역을 **덮어 씌운다.**

씌운 이후에는, **heap, stack (동적영역)**, 다른 메모리 영역이 초기화되고, OS는 그냥 실행한다. 

즉, 새로운 Process를 생성하지 않고, 현재 프로그램에 wc라는 파일을 실행한다. → 덮어쓰기

그로인해서, execvp() 이후의 부분은 실행되지 않는다. → 덮어 씌워졌기 때문

[https://fjvbn2003.tistory.com/306](https://fjvbn2003.tistory.com/306)

> **나올 수 있는 질문**
> 
> 
> System Call
> 
> - 시스템 콜에 대해 설명하세요
>     
>     System Call은 운영체제의 커널이 제공하는 서비스를 이용하기 위해 응용 프로그램의 요청에 따라 커널에 접근하기 위한 인터페이스이다. 운영체제는 커널모드(Kernel Mode)와 사용자모드(User Mode)로 나뉘어 구동되는데, 시스템 콜은 이러한 커널 영역의 기능을 사용자 모드가 사용 가능하게 한다.
>     
> - 프로세스가 종료되는 2가지 경우를 설명하세요.
>     
>     프로세스가 마지막 명령을 수행하거나 exit() 시스템 콜을 호출하였을 때 스스로 종료한다. 프로세스가 다른 프로세스를 종료시키는 경우에 부모가 자식들 중 하나의 실행을 종료할 수 있다.
>     
>     경우1. 자식이 자신에게 할당된 자원을 초과하여 사용할 때
>     경우2. 자식에게 할당된 태스크가 더 이상 필요없을 때
>     경우3. 부모가 exit를 하는데, 운영체제는 부모가 exit한 후에 자식이 실행을 계속하는 것을 허용하지 않는경우. (cascading termination(연쇄식 종료))
>