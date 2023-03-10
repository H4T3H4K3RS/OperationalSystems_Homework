Сервер:
```c++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define SHMSZ 27

void sigint_handler(int sig) {
    printf("Caught signal %d, exiting...\n", sig);
    exit(0);
}

int main() {
    signal(SIGINT, sigint_handler);

    int shmid;
    key_t key;
    char *shm, *s;

    key = 5678;

    if ((shmid = shmget(key, SHMSZ, 0666)) < 0) {
        perror("shmget");
        exit(1);
    }

    if ((shm = shmat(shmid, NULL, 0)) == (char *) -1) {
        perror("shmat");
        exit(1);
    }

    printf("Data read from shared memory: %s\n", shm);

    if (shmdt(shm) == -1) {
        perror("shmdt");
        exit(1);
    }

    printf("Shared memory segment detached\n");

    return 0;
}
```

Клиент:

```c++
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <time.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define SHMSZ 27

int main() {
    int shmid;
    key_t key;
    char *shm, *s;
    int rand_num;

    srand(time(NULL));  // инициализируем генератор случайных чисел

    key = 5678;

    if ((shmid = shmget(key, SHMSZ, IPC_CREAT | 0666)) < 0) {
        perror("shmget");
        exit(1);
    }

    if ((shm = shmat(shmid, NULL, 0)) == (char *) -1) {
        perror("shmat");
        exit(1);
    }

    s = shm;

    for (int i = 0; i < 10; i++) {
        rand_num = rand() % 100;
        printf("Generated random number: %d\n", rand_num);
        *s++ = rand_num + '0';  // записываем число в разделяемую память
    }

    *s = '\0';  // добавляем завершающий ноль

    if (shmdt(shm) == -1) {
        perror("shmdt");
        exit(1);
    }

    printf("Shared memory segment detached\n");

    return 0;
}
```
