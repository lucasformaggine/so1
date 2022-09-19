# Aula 13 - Sistemas Operacionais I

## Resumo da última aula

- Escalonamento de threads/processos
    - Processos/Threads
        - CPU Bound
        - I/O Bound
    - Métricas para avaliação de algoritmos de escalonamento
    - Custo da troca de contexto
    - Algoritmos:
        - Não preemptivos: 
        - Preemptivos
    - Algoritmos:
        - FIFO 
        - Shortest Job First (SJF)
        - SJF Preemptivo: Shortest Remaining Job First (SJRF)
        - Aula de hoje: Round Robin

## Round Robin
- Cada processo/thread tem um tempo máximo de execução contínua (time slice ou quantum)
- Duração do time slice:
    - Muito longo: se assemelha ao FIFO
    - Muito curto: multiplica-se
- Ex: comparação FIFO VS. Round Robin
    - 5 processos, que começaram nos tempos:
        - P1: 0
        - P2: 1
        - P3: 2
        - P4: 3
        - P5: 4
    - Suponha que todos os processos tem um tempo de execução de 5
    - Time slice: 1

- Round Robin:

|Processos|0  |1  |2  |3  |4  |5  |6  |7  |8  |9  |...|20 |21 |22 |23 |24 |
|---------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|P1       | x |   |   |   |   | x |   |   |   |   |   | x |   |   |   |   |
|P2       |   | x |   |   |   |   | x |   |   |   |   |   | x |   |   |   |
|P3       |   |   | x |   |   |   |   | x |   |   |   |   |   | x |   |   |
|P4       |   |   |   | x |   |   |   |   | x |   |   |   |   |   | x |   |
|P5       |   |   |   |   | x |   |   |   |   | x |   |   |   |   |   | x |

Turnaround Time
P1: (21 - 0) = 21
P2: (22 - 1) = 21
P3: (23 - 2) = 21
P4: (24 - 3) = 21
P5: (25 - 4) = 21
Média: 21

- FIFO:

|Processos|   0 - 5   |   5 - 10  |   10 - 15 |   15 - 20 |  20 - 25  |
|---------|-----------|-----------|-----------|-----------|-----------|
|P1       |     x     |           |           |           |           |           
|P2       |           |     x     |           |           |           |
|P3       |           |           |     x     |           |           |
|P4       |           |           |           |     x     |           |
|P5       |           |           |           |           |     x     |
            
Turnaround Time:

P1: 5
P2: (10 - 1) = 9
P2: (15 - 2) = 13
P4: (20 - 3) = 17
P5: (25 - 4) = 21
Média: 13

- Nesse caso, o tempo do FIFO foi menor por ser um processo puramente CPU Bound.

- Ex com 2 processos CPU Bound e 1 processo I/O Bound (o processo I/O Bound executa por 10ms e é bloqueado por 10 ms, com time slice de 100ms):

    

    |Processo    |  0 - 10  | 10 - 20 | 20 - 110 | 110 - 210 |
    |------------|----------|---------|----------|-----------|
    |I/O Bound   |    x     |         |          |           |
    |CPU Bound 1 |          |     x   |    x     |           |
    |CPU Bound 2 |          |         |          |    x      |

- Com o puro Round Robin, o processo de I/O Bound é interrompido por conta de seu bloqueio, mas nesse meio-tempo o processo CPU Bound 1 executa até o fim de seu time slice, que é por 100ms. Ao terminar o processo CPU Bound 1, por trabalhar com fila, o próximo processo a ser executado é o processo CPU Bound 2. Isso não é muito interessante, já que o processo I/O Bound foi postergado por muito tempo mesmo tendo um tempo de execução pequeno, já podendo ter sido executado antes

## Escalonamento com Prioridade

- O SO atribui um número para cada processo/thread (prioridade)
- A escolha será pelo processo/thread com maior prioridade, através de um critério lógico que evite casos de postergação desnecessária ou aumento indesejado do turnaround time
- Implementação mais eficiente do escalonamento com prioridade
- A) Múltiplas filas de processos prontos
    - Cada processo/thread entra na fila para cada prioridade. O SO vai lendo, de forma FIFO, os processos de maior priodade. Ao acabar, ele vai descendo para as prioridades menores. No entanto, se algum processo com maior prioridade entrar na fila, ele para o que está fazendo e volta à executar um processo mais prioritário

- *foto13-1*

- B) Escalonamento de loteria
    - Cada processo/thread recebe uma quantidade de bilhetes de acordo com a prioridade. Em seguida,
    o SO faz um sorteio
    - Quanto mais prioridade, mais bilhete; logo, mais chance de ser sorteado

    - No caso de múltiplas CPUs, a estrutura de dados para o escalonamento é feito por CPU (ou por um conjunto de CPUs, no caso do Windows). Porém, caso uma CPU tenha a fila vazia e a outra tenha a fila cheia de processos, a princípio não há nenhum sistema que permita que uma CPU jogue processos para a outra. Por isso, quando a quantidade de processos em cada estrutura está muito diferente, o SO migra processos de uma estrutura para a outra (problema: desperdício da cache L1)

    - Vantagens: menor chance de ter fila vazia
    - Contenção: semáforo de exclusão mútua