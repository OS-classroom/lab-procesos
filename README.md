# Procesos - Taller de laboratorio

1. Escriba en consola ```man syscalls``` y responda: ¿Qué contiene esta llamada al sistema?
2. Explique qué hace el siguiente programa:

```C

#include <syscall.h>
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>

int main(void) {
  long ID1, ID2;
  /*--------------------------------*/
  /* DIRECT SYSTEM CALL         */
  /* SYS_getpid(func no. is 20)  */
  /*--------------------------------*/
  ID1 = syscall(SYS_getpid);
  printf("syscall(SYS_getpid) = %1d\n", ID1);

  /*-----------------------------------*/
  /* "libc" WRAPPED SYSTEM CALL */
  /* SYS_getpid(func no. is 20)      */
  /*-----------------------------------*/
  ID2 = getpid();
  printf("getpid() = %1d\n", ID2);
  return 0;
}
```

