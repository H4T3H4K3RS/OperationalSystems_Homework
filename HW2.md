```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int fibonacci(int n) {
  int f0 = 0;
  int f1 = 1;
  int tmp;

  for (int i = 0; i < n; i++) {
    tmp = f0 + f1;
    f0 = f1;
    f1 = tmp;
  }

  return f0;
}

int factorial(int n) {
  int result = 1;

  for (int i = 2; i <= n; i++) {
    result *= i;
  }

  return result;
}

int main(int argc, char *argv[]) {
  int n = atoi(argv[1]);
  pid_t pid = fork();

  if (pid == 0) {
    int result = factorial(n);
    printf("Factorial: %d\n", result);
  } else {
    int result = fibonacci(n);
    printf("Fibonacci: %d\n", result);
    wait(NULL);
  }

  return 0;
}
```
