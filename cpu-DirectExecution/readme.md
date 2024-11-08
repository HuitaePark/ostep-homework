>이 숙제에서 시스템 콜과 문맥 교환의 비용을 측정할 것이다.
>
>시스템 콜의 비용을 측정하는 것은 상대적으로 쉽다.
> 
>예를들어, 간단한 시스템 콜을 (0 바이트 읽기 등) 반복적으로 호출하여 통틀어 걸린 시간을 측정한다.
>
>이 시간을 반복 횟수로 나누면 시스템 콜의 비용을 추정할 수 있다.
>
>한 가지 고려해야 할 사항은 타이머의 정확도와 정밀도이다.
>
>사용할 수 있는 전형적인 타이머는 gettimeofday()이다.
>
>자세한 사항에 대해서는 man 페이지를 참조하도록.
>
>man 페이지에서 볼 수 있는 정보는 gettimeofday()이 1970년 이후 경과한 시간을 마이크로초 단위로 반환한다는 것이다.
>
>그러나 이 사실이 타이머가 마이크로초 수준의 정밀도를 가진다는 것을 의미하지는 않는다.
>
>타이머가 얼마나 정밀한지를 알고 싶다면 gettimeofday()를 연속해서 호출하여 측정하라.
>
>이 결과로부터 만족스러운 측정 결과를 얻기 위해서는 널 시스템 콜 테스트를 몇 번 반복해야 하는지 알 수 있을 것이다.
>
>gettimeofday()가 원하는 만큼 정밀도를 보이지 않는다면 x86 CPU에서 제공되는 rdtsc 명령어를 사용하는 것을 검토해 보아야 한다.
>
>문맥 교환의 비용을 측정하는 것은 더 곤란하다.
>
>lmbench 벤치마크는 두 개의 프로세 스를 하나의 CPU에서 실행시키고 둘 사이에 Unix 파이프를 설정하여 문맥 교환 비용을 측정한다.
>
>파이프는 Unix 시스템에서 프로세스끼리 통신할 수 있는 여러 방법 중의 하나이다.
>
>첫 번째 프로세스는 첫 번째 파이프에 데이터를 쓰고, 두 번째 파이프로부터 데이터가 전달되기를 기다린다.
>
>첫 번째 프로세스가 두 번째 파이프로부터 읽을 ྕ언가를 기다린다는 사실을 알게 되면 운영체제는 첫 번째 프로세스를 봉쇄 상태로 만들고 다른 프로세스로 전환한다.
>
>이 프로세스는 첫 번째 파이프로부터 데이터를 읽고 두 번째 파이프에 데이터를 쓴다.
>
>두 번째 프로세스가 첫 번째 파이프로부터 다시 데이터를 읽으려고 하면 봉쇄 상태로 들어간다.
>
>이런 식으로 주고 받는 통신 사이클이 계속된다.
>
>이러한 통신 비용을 반복적으로 측정하여 lmbench는 문맥 교환의 비용에 대해 훌륭한 추정치를 구할 수 있다.
>
>이 숙제에서 파이프를 사용하거나 Unix 소켓과 같은 다른 통신 기법을 사용하여 유사한 상황을 만들어 볼 수 있다.
>
>문맥 교환의 비용을 측정하는 데 있어 어려움은 하나 이상의 CPU를 가진 시스 템에서 발생한다.
>
>그런 시스템에서 해야만 하는 일은 문맥 교환되는 프로세스들이 같은 프로세서 상에 위치하는 것을 보장하는 것이다.
>
>다행히도 대부분의 운영체제는 프로세스를 특정 프로세서에 ྗ어두는 시스템 콜을 가진다.
>
>예를 들어, Linux에서는 sched_setaffinity() 시스템 콜이 기대하는 일을 한다.
>
>두 프로세스를 같은 프로세서에 존재하도록 보장하면 같은 프로세서에 존재하는 한 프로세스를 중단시키고 다른 프로세스를 복원하는 운영체제의 비용을 확실하게 측정할 수 있다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <unistd.h>

long toMicroSecond(struct timeval start) {
    return start.tv_sec * 1000 * 1000 + start.tv_usec;
}

long calculateSystemCallDuration() {
    struct timeval start;
    struct timeval end;

    int fd[2];
    char c;

    pipe(fd);
    write(fd[1], "test message", sizeof("test message"));
    gettimeofday(&start, NULL);
    read(fd[0], &c, 0);
    gettimeofday(&end, NULL);

    return toMicroSecond(end) - toMicroSecond(start);
}

long calculateContextSwitchDuration() {
    struct timeval start;
    struct timeval end;

    gettimeofday(&start, NULL);
    usleep(0);
    gettimeofday(&end, NULL);

    return toMicroSecond(end) - toMicroSecond(start);
}

int main(int argc, char* argv[]) {
    printf("system call duration: %ld\n", calculateSystemCallDuration()); // 대략 0 ~ 1 마이크로초
    printf("context switch duration: %ld\n", calculateContextSwitchDuration()); // 대략 70 마이크로초
}
```
결론: 시스템 콜의 호출 비용은 매우 저렴하며 문맥 전환에는 어느정도의 비용을 지불해야 한다는 것을 알 수 있습니다.
