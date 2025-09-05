# PROCESS 

## PID
O PID é um identificador numérico exclusivo atribuído a cada processo em execução em um sistema operacional. Ele auxilia no gerenciamento e controle de processos.

## 1. Primeiro contato com `fork()`

```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main() {
    pid_t pid;

    pid = fork();

    if(pid < 0) {
        fprintf(stderr, "Fork falhou\n");
        return 1;
    }

    if(pid == 0) { // processo filho
        printf("PID do processo filho: %d\n", getpid());
        printf("PID do processo pai: %d\n", getppid());
    }
    else { // processo pai
        wait(NULL);
        printf("Processo filho terminou\n");
    }

    return 0;
}
```

### Explicação

* `fork()` duplica o processo:

  * Pai → recebe no `pid` o número do filho.
  * Filho → recebe `0`.
* `getpid()` → mostra o PID do processo atual.
* `getppid()` → mostra o PID do processo pai.
* `wait()` → faz o pai esperar o filho terminar.

**Resumo**:
Cria **2 processos** (pai e filho).
O filho imprime seus PIDs, o pai espera e depois diz que o filho terminou.

---

## 2. Usando `exec()` para rodar outro programa

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        fprintf(stderr, "Fork Failed\n");
        return 1;
    }
    else if (pid == 0) { // filho
        printf("I am the child %d\n", pid);
        execlp("/bin/ls","ls",NULL); // filho vira o "ls"
    }
    else { // pai
        printf("I am the parent %d\n", pid);
        wait(NULL);
        printf("Child Complete\n");
    }
    return 0;
}
```

### Explicação

* O filho primeiro imprime uma mensagem.
* Depois, com `exec`, ele **se transforma no programa `ls`** (lista arquivos do diretório).
* O pai espera o filho terminar.

**Resumo**:
Cria **2 processos**, mas o filho deixa de rodar o programa original e vira o `ls`.

---

## 3. Mostrando pai, filho e avô

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

#define SLEEP_TIME  2

int main() {
    pid_t pid = fork();

    if(pid < 0) {
        fprintf(stderr, "Fork faliled\n");
        return 1;
    }
    if(pid == 0) { // filho
        printf("Child process: pid = %d\n", getpid());
        sleep(SLEEP_TIME);
    }
    else { // pai
        printf("Parent process: pid do filho = %d\n", pid);
        wait(NULL);
        printf("Parent process: PID do pai = %d\n", getpid());
        printf("Parent process: PID do avô = %d\n", getppid());
    }
    return 0;
}
```

### Explicação

* Mostra o **PID do filho** (`getpid`).
* O pai mostra o **PID dele mesmo** e o **PID do avô** (`getppid` → geralmente o terminal/bash).

**Resumo**:
Dá pra ver claramente a hierarquia: **avô → pai → filho**.

---

## 4. Vários forks em sequência

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    fork();
    fork();
    fork();

    printf("PID = %d\n", getpid());
    return 1;
}
```

### Explicação

* Cada `fork()` duplica os processos existentes.
* Com 3 `fork()`, o total final é **2³ = 8 processos**.
* Cada um imprime seu PID.

**Resumo**:
Esse tipo de código gera uma **árvore cheia de processos**.
Total final: **8 processos** (7 criados + 1 original).

---

## 5. Criando irmãos (siblings)

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    for(int i=0; i<5; i++) { 
        if(fork() == 0) {
            printf("[child] pid %d from [parent] pid %d\n", getpid(), getppid());
            exit(0);
        }
    }
    
    for(int i=0; i<5; i++) 
        wait(NULL);
}
```

### Explicação

* O pai cria **5 filhos** em um loop.
* Cada filho imprime seu PID e o do pai.
* `exit(0)` evita que os filhos também entrem no loop.
* O pai espera todos terminarem.

**Resumo**:
Temos **5 filhos irmãos (siblings)**, todos com o mesmo pai.

---

## 6. Forks com condições

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    pid_t c1;
    pid_t c2 = 0;

    c1 = fork(); // fork 1
    if(c1 == 0)
        c2 = fork(); // fork 2
    fork(); // fork 3 - numero de processos dobram
    if(c2 > 0)
        fork(); // fork 4 - apenas os processos derivados do fork 2 podem ter c2 > 0

    return 0;
}
```

### Explicação

* `fork 1` → cria P1.
* `fork 2` (só no filho P1) → cria P2.
* `fork 3` (todos executam) → duplica os existentes.
* `fork 4` (só onde `c2 > 0`, ou seja, P1 e cópias dele) → cria mais 2 processos.

**Resumo**:
No final existem **8 processos**.
Nem todos executam todos os forks → depende das condições (`if(c1==0)` e `if(c2>0)`).

---

# Dicas pra prova

* `fork()` duplica o processo.
* Sempre pense: **quantos processos existiam antes** × 2 depois do fork.
* Use condições (`if`) para saber **quem continua criando**.
* `exec()` substitui o código do filho por outro programa.
* `wait()` impede processos zumbis.
* `getpid()` = PID atual.
* `getppid()` = PID do pai.
