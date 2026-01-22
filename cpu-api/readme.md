# 문제

**1. fork()를 호출하는 프로그램을 작성하라.
fork()를 호출하기 전에 메인 프로세스는 변수에 접근하고 (예, x) 변수에 값을 지정하라 (예, 100).
자식 프로세스에서 그 변수의 값은 무엇인가?
부모와 자식이 변수 x를 변경한 후에 변수는 어떻게 변했는가?**

```c
#include <stdio.h>     
#include <unistd.h>    
#include <sys/wait.h>  
#include <stdlib.h>     
int main(int argc, char *argv[])
{
    int status = fork();
    int x = 100;
    if(status <0){
        perror("Fork failed");
        exit(1);
    }
    else if(status == 0){
        printf("Child Process: PID = %d, Parent PID = %d\n", getpid(), getppid());
        x += 50;
        printf("x in Child Process: %d\n", x);
        exit(42);
    }
    else{
        printf("Parent Process: PID = %d, Child PID = %d\n", getpid(), status);
        x -= 50;
        printf("x in Parent Process: %d\n", x);
    }
}
```
각자 다른 변수 값을 가진다  
<img width="360" height="68" alt="image" src="https://github.com/user-attachments/assets/4939d551-2aec-4e4c-ac45-3593edcf4a65" />

 

**2. open() 시스템 콜을 사용하여 파일을 여는 프로그램을 작성하고 새 프로세스를 생성하기 위하여 fork()를 호출하라.
자식과 부모가 open()에 의해 반환된 파일 디스크립터에 접근할 수 있는가?
부모와 자식 프로세스가 동시에 파일에 쓰기 작업을 할 수 있는가?**

 
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/wait.h>
int main(){
     int fd = open("test.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
     
     if (fd < 0) {
        perror("open");
        return 1;
    }
    int rc = fork();
    if (rc == 0) {
        write(fd, "Child writing to file.\n", 23);
    }
    else {
        wait(NULL);
        write(fd, "Parent writing to file.\n", 24);
    }
    close(fd);
    return 0;
}

```
결과:  
<img width="247" height="43" alt="image" src="https://github.com/user-attachments/assets/d9a37948-f439-4d72-affa-36f4728a592a" />  
부모와 자식이 동시에 접근 가능함. 파일 디스크럽터는 공유 가능한 자원임
 

**3. fork()를 사용하는 다른 프로그램을 작성하라.
자식 프로세스는 “hello”를 출력하고 부모 프로세스는 “goodbye”를 출력해야 한다.
항상 자식 프로세스가 먼저 출력하게 하라.
부모가 wait()를 호출하지 않고 할 수 있는가?**

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
int main(){
    int fd[2];
    pipe(fd);

    int rc = fork();
    if(rc==0){
        printf("hello\n");
        write(fd[1], "hello", 6);
    }
    else{
        char buf;
        read(fd[0], &buf, 6);
        printf("goodbye\n");
    }
}
```
pipe를 사용하면 가능  
<img width="102" height="37" alt="image" src="https://github.com/user-attachments/assets/676f4475-afb1-4342-b203-309b6d8874e2" />
자식이 write를 할동한 부모는 block , 끝나면 read를 하게 된다.
 

**4. fork()를 호출하고 /bin/ls를 실행하기 위하여 exec() 계열의 함수를 호출 하는 프로그램을 작성하라.
exec()의 변형 execl(), execle(), execlp(), execv(), execvp(), execve() 모두를 사용할 수 있는지 시도해 보라.
기본적으 로는 동일한 기능을 수행하는 시스템 콜에 여러 변형이 있는 이유를 생각해 보라.**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
int main(){
    int rc = fork();
    if(rc==0){
        execl("/bin/ls", "ls", "-l", NULL);
    }
    else if(rc>0){
        wait(NULL);
        printf("Child Complete\n");
    }
    return 0;
}
```
<img width="531" height="201" alt="image" src="https://github.com/user-attachments/assets/38eddc2b-1193-41b1-97be-9f51f442b303" />  
추가적으로 인자나 환경변수를 전달하기 위해 존재하는 것 같다.

 

**5. wait()를 사용하여 자식 프로세스가 종료되기를 기다리는 프로그램을 작성하라.
wait()가 반환하는 것은 무엇인가? 자식 프로세스가 wait()를 호출하면 어떤 결과가 발생하는가?**
```c
#include <stdio.h>     
#include <unistd.h>    
#include <sys/wait.h>  
#include <stdlib.h>     
int main()
{
    int status = fork();
    if(status <0){
        perror("Fork failed");
        exit(1);
    }
    else if(status == 0){
        printf("Child Process: PID = %d, Parent PID = %d\n", getpid(), getppid());
        exit(42);
    }
    else{
        int waitId = wait(NULL);
        printf("Parent Process: PID = %d, Child PID = %d\n", getpid(), status);
        printf("Wait returned PID = %d\n", waitId);
    }
}
```
wait()이 반환하는 값은 종료된 자식의 id이다.  
<img width="373" height="45" alt="image" src="https://github.com/user-attachments/assets/c2b9662c-3308-470d-b582-7be6197d54f3" />  
만약 wait을 자식에게 쓴다면 부모가 종료되길 기다리는 자식이 된다.  
<img width="345" height="40" alt="image" src="https://github.com/user-attachments/assets/f8b2dd82-33e5-490b-9022-5f563aadaee7" />



**6. 위 문제에서 작성한 프로그램을 수정하여 wait() 대신에 waitpid()를 사용하라.
어떤 경우에 waitpid()를 사용하는 것이 좋은가?**
```c
int waitId = waitpid(status, &x, 0);
```
이런식으로 기다릴 자식을 정할수도, 심지어는 안기다릴수도 있다.선택하는 경우에 유용할것 같다.

 

**7. 자식 프로세스를 생성하고 자식 프로세스가 표준 출력 (STDOUT_FILENO)을 닫는 프 로그램을 작성하라.
자식이 설명자를 닫은 후에 아무거나 출력하기 위하여 printf() 를 호출하면 무슨 일이 생기는가?**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <fcntl.h>

int main(int argc, char *argv[]) {

    int rc = fork(); // 자식 프로세스 생성

    if (rc == 0) { 
        printf("자식: 표준 출력을 닫기 전입니다.\n");
        close(STDOUT_FILENO); 
        int result = printf("자식: 표준 출력을 닫은 후입니다. 이 메시지가 보이나요?\n");
        if (result < 0) {
            fprintf(stderr, "자식: printf 호출 실패 (반환값: %d)\n", result);
            perror("자식: 오류 상세 내용");
        } else {
            fprintf(stderr, "자식: printf 호출 성공 (반환값: %d)\n", result);
        }
    } 
    else {
        int wc = wait(NULL); 
        printf("부모: 자식 프로세스(%d)가 종료되었습니다.\n", rc);
    }

    return 0;
}
```
<img width="316" height="67" alt="image" src="https://github.com/user-attachments/assets/c0490544-3a48-4b63-982d-8b42b9188f83" />  
출력이 보이질 않는다.
또한 에러가 발생하지만 그 또한 출력되진 않는다.



**8. 두 개의 자식 프로세스를 생성하고 pipe() 시스템 콜을 사용하여
한 자식의 표준 출력을 다른 자식의 입력으로 연결하는 프로그램을 작성하라.**
3번 참조
