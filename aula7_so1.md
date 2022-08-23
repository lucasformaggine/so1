# SO1 - Aula 7

## Resumo da última aula
- Estados de um processo
    - Executando (+1 CPU, +1 processo pode ser executado simultaneamente, um de cada CPU)
    - Bloqueando
    - Pronto
- Process Control Block (PCB)
    - Estrutura de dados que guarda as informações de um processo
    - Campos importantes:
        - Número do processo (PID) (P)
        - Número do processo pai (P)
        - Valor dos registradores (T)
        - Estado (T)
        - Localização das áreas de memória do processo (P)
        - Informações para escalonamento (por ex: prioridade) (T)
        - Informações sobre recursos externos sendo utilizados (P)
        - Código de retorno (P)
        - Diretório atual (P)
        - Diretório raiz (P)
- Troca de contexto
    - Um processo deixa de ser executado e outro passa a ser executado em seu lugar, por algumas razões; através de uma interrupção, o SO salva o contexto do processo em execução através de registradores, muda seu estado e coloca outro processo para executar através do escalonamento de processos restaurando seus registradores
- Threads
    - Uso para processos com mais de uma ação simultânea (ex: um processo realiza a interação com o usuário e o outro realiza cálculos; isso ocorre através de pontos de execução em trechos de um mesmo programa)
    - Criação, término
    - Espera pelo término

## Diagrama ER das threads e processos
- Convencional: Processo (1) <-> (n) Threads
. Linux: PCB Thread 1 <- (Áreas da Memória) (Arquivos em Uso)-> PCB Thread 2
- Isto é: no Linux, os PCBs são exclusivamente para threads; logo, informações compartilhadas entre ambas são simultaneamente apontadas na memória
- Em outros sistemas operacionais, os processos possuem PCBs (com informações de processo) e as threads possuem PCBs como "TCBs" (isto é, informações ora de processo, ora de thread)

## Condição de corrida (Race condition)

- Usos simultâneos de memória em diferentes threads de um processo

- Exemplo com 1 CPU:
    - Saldo com valor 500 na posição 5000 da memória

Thread 1:
```asm
mov eax, [5000]
add eax, 100
mov [5000], eax
```
Thread 2:
```asm
mov ebx, [5000]
add ebx, 300
mov [5000], ebx
```

- Supondo que a thread 1 é executada primeiro, depois a thread 2 e por último a thread 1. O valor resultado só será a conta da última thread executada (isto é, a thread 1), já que os registradores são sempre restaurados (ou outros são utilizados) durante as trocas de contexto. Logo, começando com 500, o valor final seria 600. Caso a thread 2 fosse a última a ser executada, então o valor final seria 800. No entanto, o valor esperado seria 900.
- Causa do problema: trecho crítico (os trechos de add)
- Apenas um processo ou thread pode executar um trecho crítico (exclusão múltipla)
- Um processo ou thread fora do seu trecho crítico não pode impedir que ourto processo ou thread entre no seu trecho crítico (progresso)
- Nenhum processo ou thread pode esperar um tempo arbitrariamente longo para entrar no seu trecho crítico (espera limitada)

- Solução 1: via hardware
    - Durante a execução de uma thread, antes de toda a execução por trás do trecho crítico, desabilitar as interrupções da CPU
    - As interrupções só são reabilitadas após a execução do trecho crítico
    - Problemas: a instrução de desabilitar interrupções possui privilégios, portanto necessitaria modo de operação privilegiado; com máquinas com mais de uma CPU, a desabilitação só vai ocorrer para a CPU que está executando o código de uma determinada thread, portanto não resolverá

- Solução ingênua:

  Thread 1 
    
```c
    while (vez != 1)     // Inicio do TC
    saldo = saldo + 100; // Trecho critico
    vez = 2;             // Fim do TC
```

  Thread 2
  
```c
    while (vez != 2)     // Inicio do TC
    saldo = saldo + 300; // Trecho critico
    vez = 1;             // Fim do TC
```

- Problema: Eu não posso executar uma mesma thread 2 vezes seguidas.

- Solução Genérica:

- O vetor interessado se refere ao interesse da thread de entrar no trecho crítico. 

Thread 1

```c
    interessado[1] = true;
    vez = 2;

    while (interessado[2] && vez == 2);

    saldo = saldo + 100;
    interessado[1] = false;
```

Thread 2

```c
    interessado[2] = true;
    vez = 1;

    while (interessado[1] && vez == 1);

    saldo = saldo + 300;
    interessado[2] = false;
```

- Limitação: A implementação não é muito boa para muitas threads

### Instrução Atômica

- Instrução em Linguagem de Máquina: 

1) Test and Set
2) Swap

```as
    tsl eax, [2000]
```
(Na realidade a CPU Intel não tem test and set, ela tem swap)


- Implementação em C:

```c
int testAndSet (int p_valor) {
    int ret = *p_valor;
    *p_valor = true;
    return ret;
}
```
p_valor indica se está no trecho crítico ou não

Thread 1

```c
    while(testAndSet(&lock)); // Inicio do TC
        /* não faz nada */

    saldo = saldo + 100;

    lock = false; // Fim do TC
```

Thread 2

```c
    while(testAndSet(&lock));

    saldo = saldo + 300;

    lock = false;
```

- O TSL atualiza o valor na memória, e depois atualiza o valor antigo no registrador


- Todas as soluções acima tem um problema:
    - Gasto de CPU executando o loop, enquanto a thread não entra no trecho crítico.

- Mecanismo de Loop: Spin Lock