# Aula 12 - Sistemas Operacionais I

## Resumo da última aula
- Deadlock: um conjunto de threads está esperando por um evento que apenas pode ser gerado por outra thread no conjunto.
- Condições necessárias do deadlock:
    1) Exclusão mútua no acesso ao recurso
    2) Hold and wait
    3) Não preempção do recurso
    4) Espera circular entre as threads
- Abordagens:
    1) Não fazer nada automatizado
    2) Prevenção: eliminar uma das 4 condições
    3) Evitar deadlocks
    4) Detectar e eliminar deadlocks

## Escalonamento da CPU
### Introdução
- É a decisão do SO para qual processo/thread será executada de acordo com uma prioridade
- Algoritmos cada vez mais eficientes para realizar o escalonamento
- SO guarda threads (TCBs) em diferentes filas
    - Fila das threads prontas
    - Threads esperando por E/S de um dispositivo
    - Threads esperando em alguma primitiva de sincronização (que bloqueia a thread)
- Threads não são iguais
- Algumas threads usarão a CPU de forma intensa ao longo do tempo (CPU Burst ou Surto de CPU), intervalando pela espera de E/S de um dispositivo (CPU Bound Thread ou Thread Limitada pela CPU)
- Algumas threads usarão a CPU de forma mais branda, porém ficarão em grandes intervalos de tempo dependendo da E/S (I/O Bound Thread ou Thread Limitada pela E/S)

### Job
- É uma tarefa que precisa da CPU durante um tempo
- Tempo de excecução: tempo total que o job usoua CPU (isto é, não inclui o tempo em que o processo aguarda por uma E/S, por exemplo)
- Momento de submissão / Momento de chegada: momento em que o job foi submetido para a execução
- Deadline: momento em que o job precisa ter sua tarefa realizada
- Turnaround time ("tempo total"): soma do tempo de execução com o tempo de espera
- Throughput: jobs terminados por unidade de tempo
- Preocupações para ter um bom escalonamento:
    - Justiça: distribuir a CPU da forma mais igualitária possível
    - Starvation: esperar muito para utilizar o recurso que se deseja
    - Overhead: quanto tempo não está sendo executado código dos jobs (para uma CPU); isto é, durante uma troca de contexto, para que execute o algoritmo de escalonamento
    - Previsibilidade: comportamento do algoritmo de escalonamento de ser parecido em situações similares
- De foram geral, os sistemas priorizam a justiça.
- Em sistemas batch, onde não há interação com o usuário, a prioridade é maximizar o throughput, diminuir o turnaround e aumentar o uso da CPU
- Em sistemas interativos, a preocupação é reduzir o tempo de resposta
- Em sistemas de tempo real, a preocupação é atingir os deadlines e a previsibilidade

### Overhead
- Tempo gasto ("custo") para preservar os registradores em uso do job que está saindo de execução
- Tempo gasto para execução de um algoritmo 
- Tempo gasto para restaurar os registradores do job que vai entrar em execução
- Tempo gasto consequente pelo esvaziamento da cache (L1, L2, L3 e TLB; no caso da última, é uma cache de controle da memória), pelo hardware. Logo no primeiro acesso, como a cache está vazia, o próximo processo faz a primeira execução com acesso à memória

### Algoritmo de Escalonamento
- O algoritmo de escalonamento executa de modo não preemptivo:
    1) Quando um job é bloqueado (por E/S ou por primitiva de sincronização)
    2) Quando o job termina
    3) Quando o job "libera" a CPU
    - As condições correspondem a ações que o job executou
- O algoritmo de escolamento executa de modo preemptivo:
    1) Quando terminar o time slice de um job, logo o próximo precisa ser escalonado (interrupção por relógio)
    2) Quando houver outras interrupções, que desbloqueem outro job
    - As condições cporresndem a ações que o SO executou
- Algoritmo FIFO: first in, first out (fila)
    - Exemplo:
    - Processos P1, P2 e P3: tem um tempo total durante a execução (turnaround time)
    - tt(P1) = 12; tt(P2) = 3; tt(P3) = 3
    - Caso 1: ordem de chegada P1, P2, P3 (*foto12-1*)
        - Tempo total P1: 12
        - Tempo total P2: 15
        - Tempo total P3: 18
        - Tempo total total: 45
        - Tempo total médio: 15
        - Pode-se perceber que o tempo de espera total é menor. Nesse caso, é 27
    - Caso 2: ordem de chegada P2, P1, P3
    (*foto12-2*)
        - Tempo total P2: 3
        - Tempo total P3: 6
        - Tempo total P1: 18
        - Tempo total total: 27
        - Tempo total médio: 9
        - Pode-se perceber que o tempo de espera total é menor. Nesse caso, é 9
- Algoritmo "Shortest Job First" (SJF)
    - Para SOs com execução de vários processos (SO multiprogramado)
    - O algoritmo pode utilizar a previsão de qual thread terá o menor tempo de CPU burst, porém é difícil prever este tempo
    - Pode ser preemptivo, na possibilidade de poder fazer troca de contexto (sistemas atuais)
        - Também chamado de "Shortest Remaining Job First" (SRJF)
        - Caso um job submetido tenha um tempo de execução menor que o job atualmente executado, então o novo job passa a executar
        - Exemplo:

        Job|Momento de chegada|Burst time
        ---|------------------|----------
        J1 |0                 |8
        J2 |1                 |4
        J3 |2                 |9
        J4 |3                 |5

        - SJF normal (não preemptivo): o J1 inicialmente roda sem escolha, já que é o primeiro da fila; no entanto, até o fim de sua execução, é possível realizar o cálculo para prever o burst time dos outros jobs. Nesse sentido, o próximo job a ser executado é o J2, em seguida o J4 e por fim o J3.
        - *foto12-4*
        - Dado que J2 começou apos 1 unidade de tempo, J3 após 2 unidades e J4 após 3 unidades, então segue abaixo o tempo para começar a executar (tempo de espera):
        
        Jobs | Tempo de espera
        -----|----------------
        J1   |0
        J2   |7
        J3   |9
        J4   |15

        - Tempo médio de espera: (7 + 9 + 15)/4 = 7,75
         
        - SJF preemptivo: o J1 começa rodando sem escolha por 1 unidade de tempo, porém no momento que o J2 entra na fila, existe uma preempção buscando aquele com o menor tempo restante; então, o J2 entra em execução; ele roda por mais uma unidade de tempo até o momento que o J3 entra na fila, e como o J2 continua sendo aquele com o menor tempo restante, ele segue executando por mais 1 unidade de tempo; então, J4 entra na fila e ocorre mais uma busca pelo menor tempo, que segue sendo o de J2; J2 é finalizado e então dentre os restantes J4 é aquele com menor tempo restante. Ele é executado até o fim, até a mesma situação ocorrer com J1, e por fim com J3. Logo, a ordem de execução foi: J1(1), J2(4), J4(5), J1(7), J2(9)
        - *foto12-5*

        Tabela dos jobs em relação aos tempos que faltam para cada instante de tempo

        Job |t=1|t=2|t=3|t=5|t=10
        ----|---|---|---|---|----
        J1  | 7 | 7 | 7 | 7 | 7
        J2  | 4 | 3 | 2 | x | x
        J3  | x | 9 | 9 | 9 | 9
        J4  | x | x | 5 | 5 | x

        - Tempo para começar a executar (tempo de espera):
            - J2 = 0
            - J3 = 17 - 2 = 15
            - J1 = 0 + 9 = 9
            - J4 = 2
            - Total = 0 + 15 + 9 + 2 = 26
            - Média = 6,5

### Round Robin

- "Time slice" para a execução de jobs
- Ocorre escalonamento quando um job alcança um tempo de execução (em um CPU burst)
igual ao time slice
- "Qual é o bom time slice?"
    - Muito grande: FIFO
    - Muito curto: muito overhead
    - Algo na média consegue dar um bom tempo de resposta, bom tempo de execução, mas pouco gasto de tempo com esvaziamento de cache


    