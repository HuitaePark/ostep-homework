```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/time.h>
#include <sys/syscall.h>

// 반복 횟수
#define LOOPS 1000000

long get_elapsed_usec(struct timeval start, struct timeval end) {
    return (end.tv_sec - start.tv_sec) * 1000000 + (end.tv_usec - start.tv_usec);
}

int main() {
    struct timeval start, end;
    long elapsed;
    int i;
    
    // ==========================================
    // 시스템 콜 비용 측정
    // ==========================================
    printf("Measuring System Call Cost...\n");
    
    gettimeofday(&start, NULL);
    for (i = 0; i < LOOPS; i++) {
        read(STDIN_FILENO, NULL, 0); 
    }
    gettimeofday(&end, NULL);

    elapsed = get_elapsed_usec(start, end);
    printf("Total time for %d syscalls: %ld us\n", LOOPS, elapsed);
    printf("Average time per syscall: %f us\n\n", (float)elapsed / LOOPS);


    // ==========================================
    // 문맥 교환 비용 측정
    // ==========================================
    printf("Measuring Context Switch Cost...\n");

    int pipe1[2], pipe2[2];
    char buf = 'x';

    if (pipe(pipe1) < 0 || pipe(pipe2) < 0) {
        perror("pipe");
        exit(1);
    }

    pid_t rc = fork();

    if (rc < 0) {
        perror("fork");
        exit(1);
    } else if (rc == 0) {
        // [자식 프로세스]
        for (i = 0; i < LOOPS; i++) {
            read(pipe1[0], &buf, 1);
            write(pipe2[1], &buf, 1);
        }
        exit(0);
    } else {
        // [부모 프로세스]
        gettimeofday(&start, NULL);
        for (i = 0; i < LOOPS; i++) {
            write(pipe1[1], &buf, 1);
            read(pipe2[0], &buf, 1);
        }
        gettimeofday(&end, NULL);

        elapsed = get_elapsed_usec(start, end);
        printf("Total time for %d loop iterations: %ld us\n", LOOPS, elapsed);
        printf("Average time per Context Switch: %f us\n", (float)elapsed / (LOOPS * 2));
    }

    return 0;
}
```
<img width="381" height="113" alt="image" src="https://github.com/user-attachments/assets/972277d0-7df7-4f48-9c36-fe1c9c612edd" />

