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
3. Ejecute el comando ```man execl```. Liste las funciones e indique qué hace cada una de ellas.
4. Compile y ejecute el siguiente programa:

```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main(void) {
	int fd;
	pid_t pid;
	int num;
	if ((pid = fork()) < 0) {
		perror("fork falló");
		exit(-1);
	} else if (pid == 0) {
		for (num=0; num<20; num++) {
			printf("hijo: %d\n", num);
                        sleep(1);
		}
	} else {
		for (num=0; num<20; num+=3) {
			printf("padre: %d\n", num);
                        sleep(1);
		}
	}
}
```
**Responda**:
* ¿Qué significa el retorno de la función fork?
* ¿Cuál es la salida esperada en pantalla?
* ¿Cómo es posible que la sentencia printf reporte valores diferentes para la variable num en el hijo y en el padre?

5. Modifique los Códigos del 3 al 8 (añadiendo donde sea necesario un llamado a wait) para que los programas se ejecuten como realmente se esperaría.
6. Dado el siguiente código:

```C
#include<stdio.h>
#include<unistd.h>
main() {
   printf("Hola ");
   fork();
   printf("Mundo");
   fork();
   printf("!");
}
```
**Resuelva**:
* Sin ejecutarlo dibuje la jerarquía de procesos del programa y determine cuál es la posible salida en pantalla.
* Compile y ejecute el programa. ¿Es la salida en consola la que usted esperaba? ¿Cuál puede ser la razón de esto? (ayuda: función fflush: fflush(stdout);)
* Modifique el programa de tal manera que se creen exactamente 3 procesos, el padre imprime "Hola", el hijo imprime "Mundo" y el hijo del hijo imprime "!", exactamente en ese orden.

7. Dado el siguiente código:

```C

#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/wait.h>
main() {
  pid_t  pid;
  int status;
  printf("PADRE: Mi PID es %d\n", getpid());
  printf("PADRE: El PID de mi padre es %d\n", getppid());
  pid = fork();
  if(pid == 0){
    sleep(5);
    printf("HIJO: Mi PID es %d\n", getpid());
    printf("HIJO: El PID de mi padre es %d\n", getppid());
    printf("HIJO: Fin!!\n");
  }
  else {
    printf("PADRE: Mi PID es %d\n", getpid());
    printf("PADRE: El PID de mi hijo es %d\n", pid);
    // wait(&status);
    // printf("PADRE: Mi hijo ha finalizado con estado %d\n", status);
    printf("PADRE: Fin!!\n");
  }
  exit(0);
}
```
**Responda**:
* ¿Cuál es la principal función de sleep en el código anterior?
* ¿Quién es el padre del padre? Use este comando: 	

```
ps -alf
```

* ¿Por qué el proceso hijo imprime el id del padre como 1? ¿Es el que usted espera de acuerdo la jerarquía de procesos?
* Retire el comentario de las líneas de la función wait y la siguiente función printf. ¿Cuál es el identificador del padre ahora? ¿Para qué sirve la función wait? ¿Qué retorna en status? 

8. Proceso zombie:

```C
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
  pid_t pid;
  char *message;
  int n;
  printf("Llamado a fork\n");
  pid = fork();
  switch(pid) {
    case -1:
      perror("fork falló");
      exit(1);
    case 0:
      message = "Este es el hijo";
      n = 1;
      break;
    default:
      message = "Este es el padre";
      n = 30;
      break;
  }
  for(; n > 0; n--) {
    printf("n=%d ",n);
    puts(message);
    sleep(1);
  }
  exit(0);
}
```

Cuando un proceso hijo termina, su asociación con el padre continúa mientras el padre termina normalmente o realiza el llamado a wait. La entrada del proceso hijo en la tabla de procesos no es liberada inmediatamente. Aunque el proceso no está activo el proceso hijo reside aún en el sistema porque es necesario que su valor de salida exista en caso de que el proceso padre llame wait. Por lo tanto él se convierte en un proceso zombie.

* Realice el comando  ps –ux  en otra terminal mientras el proceso hijo haya finalizado pero antes de que el padre lo haga. ¿Qué observa en las líneas de los procesos involucrados?
* ¿Qué sucede si el proceso padre termina de manera anormal?

9. Familia de funciones execl: execl, execlp, execle, exect, execv y execvp y todas las que realizan una función similar empezando otro programa. El nuevo programa empezado, sobrescribirá el programa existente, de manera que nunca se podrá retornar al código original a menos que la llamada execl falle.

* **Programa 1**:

```C
#include <unistd.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/wait.h>
main()
{
  int pid;
  if ((pid = fork()) == 0) {
  		execl("/bin/ls", "ls", "/", 0);
  }
  else {
    wait(&pid);
    printf("exec finalizado\n");
  }
}
```

* **Programa 2**:

```C

#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
int main()
{
  printf("Corriendo ps con execlp\n");
  execlp("ps", "ps", "-ax", 0);
  printf("Echo.\n");
  exit(0);
}
```
**Resuelva**:
* ¿Qué es lo que hace cada uno de los programas anteriormente mostrados?

10. Haga un programa que cree 5 procesos donde el primer proceso es el padre del segundo y el tercero, y el tercer proceso a su vez es padre del cuarto y el quinto.

El programa debe tener la capacidad de:
* Verificar que la creación de proceso con fork haya sido satisfactoria.
* Imprimir para cada proceso su id y el id del padre.
* Imprimir el id del proceso padre del proceso 1.
* A través de la función system imprimir el árbol del proceso y verificar la jerarquía (pstree).

11. Codifique un programa que haga lo siguiente:

12. En construccion

13. En construccion


**Referencias**:
En construccion






