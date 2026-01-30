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
어느정도의 마이크로초를 소비한다.

