## Resumo da última aula

### Principais cmadas do SO relacionadas a processos
- Auto término
- Término do processo
- Criação de processos
    - Mudança de prioridade
    - Arquivos iniciais
    - Diretório corrente de um processo
    - CHamada fork no Linux: sem parâmetro

    Upon successful completion, fork() returns 0 to the child process and returns the process ID of the child process to the parent process.

    - Chamada de término do processo:
        - 0 -> sem problema algm
        - Diferemte de 0 -> algm tipo de erro

    - Janela de espera: pai esperar pelo filho

## Estados de um Processo

- Exemplo: dados 3 processos p1, p2 e p3, antes de executar, estão em um estado pronto (t1)
    - Ao tentar iniciar os processos que estão em um estado pronto, o SO escolhe, através do escalonamento de processos, um deles para ser executado. Esse processo escolhido estará em um estado de execução (digamos:  p2) (t2)
    - Uma máquina com uma única CPU só pode executar um processo por instante de tempo. No caso de um multicore, ela pode executá-los simultaneamente, um estando em cada core
    - Supondo que p2 pede ao usuário algum input. Enquanto o usuário não fizer esse input, o programa precisa aguardar. Para não gargalar o fluxo, ele vai para um estado bloqueado (t3)
    - Para não deixar o estado de execução vazio, o SO faz um novo escalonamento de processos e escolhe, dentre as prioridades, o p1 ou o p3 para ser executado (digamos: p3). Esse processo escolhido irá para o processo de execução (t4)
    - Caso o p3 peça ao SO para receber dado de outros processos; de forma análoga ao ocorrido em t3, enquanto o dado não chegar ao p3, esse processo irá para o estado de bloqueado (t5)
    - Nesse momento, o SO executa o p1, que é o único processo restante do estado de prontidão. Portanto, p1 vai para o estado de execução (t6)
    - Caso o usuário tenha digitado o dado que p2 precisava, ele volta para o estado pronto, porém não é executado pois já existe um processo no estado de execução (no caso, p1) (t7)
    - Como o p2 tem interação com o usuário, seria mais interessante que ele fosse executado logo. Porém, supondo que sua prioridade é menor, há duas soluções possiveis:
        - 1) O p1 faz uma chamada ao SO para ceder a vez através de uma estrutura programável. Nesse caso, ele volta para o estado pronto (t8). Então, o SO consegue enviar o p2 para o estado pronto (t9)
        - 2) O SO gera um tempo limite para cada processo execuar (time-slice ou time-quantum). Quando o limite estoura, o SO tira de execução o processo e o envia para o estado de execução. Esse comportamento é chamado de preempção (preemption), nesse caso preempção por temp (t8). Então, o SO consegue enviar o p2 para o estado pronto (t9)
    - Quando um processo envia dados para p3, o mesmo deixa o estado bloqueado e vai ao estado pronto (t10)
    - Supondo que p3 possui alta prioridade, maior que a de p2, então o SO opta por interromper a execução de p2 e trocá-los de estados, através de uma preempção por prioridade (t11)
- No Linux, há estados a mais
- No Linux, ocorrem "estados zumbis", no qual o pai não faz retorno para o processo filho, logo este não consegue mais se comunicar
- O bloqueio de processo também pode ocorrer por outros fatores além desses dois exemplos.
    - Leitura de algo em disco
    - Espera por dados de outro computador
    - Relógio digital (não é por falta de dado, e sim aguardando 'passar o tempo')

## Process Control Block (PCB)
- É uma estrutura de dados que se localiza na memória do SO (através de um array de PCBs), servindo para guardar informações do processo, como:
    - Número do processo (PID)
    - Número do processo pai (PID do pai)
    - Valores dos registradores (para processar os valores caso eles tenham sido interrompidos, permitindo que eles sejam restaurados aos registradores com seu retorno à execuçã - importante ressaltar o registrador que guarda o endereço da instrução em execução)
    - Estado do processo
    - Localização das áreas de memória do 
    - Informações para escalonamento (ex: prioridade)
    - Informações sobre recursos externos em uso (exemplo: arquivos de uso)
    - Código de retorno
    - Diretório corrente do processo
    - Diretório raiz do processo (imporante para containerização)
    - Ponteiros para outros PCBs (uso interno)
## Troca de contexto
- Exemplo: suponha que um processo p1 está executando, mas por limite de tempo, ele para de rodar na linha X

### p1
- linha 1
- linha 2
- linha 3
- ...
- linha x-1
- linha x --> time preemption; o SO preserva o valor dos registradores no PCB do p1 e altera o estado do p1 para o estado pronto, altera o estado do p2 para o estado de execução e reserva o valor dos registradores para o processo p2

### p2
- linha y
- linha y+1
- ...
- linha z-1
- linha z --> time preemption; o SO preserva o valor dos registradores no PCB do p2 e altera o estado do p2 para o estado pronto, altera o estado do p1 para o estado de execução e reserva o valor dos registradores para o processo p1

### p1
- linha x+1
- linha x+2
- ...

- O processo de troca de contexto envolve armazenar dados dos registradores, reservá-los e trocar processos de estados
- Além disso, a memória também precisa ser reconfigurada durante o processo. Com o uso de threads, não é necessário reconfigurá-la.

## Thread

- Motivações:
    - Troca de contexto: é possível otimizá-la?
    - a) como consigo parar um cálculo que um processo está fazendo sem matar o processo?
    - b) como consigo fazer uma consulta a um servidor remoto e ao mesmo tempo interagir com o usuário?
- Possíveis soluções ("iniciais"): 
    - Solicitar ao SO o conteúdo de várias "entradas" diferentes (isto é: ao pedir dado do SO, ele vem de alguma fonte; em uma única chamada, é possível solicitar esses dados e, o primeiro que estiver disponível, será retornado nessa chamada). Isso é chamado de multiplicação da E/S, e resolve parcialmente o caso "b", por exemplo pela chamada "select" do Unix. 
    - Para que o caso seja garantidamente resolvido, é necessário o uso de chamadas assíncronas, já que caso o SO não encontre o dado, ele não bloqueia o processo, porém o SO retorna imediatamente essa informação para o que quer ser executado. Dessa forma, trata o caso b e também o caso a, pois durante o cálculo essas "perguntas" (através das chamadas assíncronas) verificam se outro processo pode ser executado; caso não, o cálculo continua
- Da forma convencional, a resolução é um pouco complicada. Por isso, vale a pena o uso de thread
- Existe mais de um ponto de execução nos processos; no caso de computadores com multicore, cada núcleo trata de uma thread, e portanto os contextos agem de forma paralela. Exemplo: em um mesmo processo, um ponto de execução aguarda o resultado do usuário, enquanto outro executa o cálculo;
- Exemplo (sem thread):
```c
long int somaTudo() {
    int i;
    long int soma = 0;

    for (i = 1; i <= 10000000; i++) {
        soma += i;
    }

    return soma;
}

int main() {
    printf("O valor da soma é %l\n", somaTudo());
    return 0;
}
```
- Nesse exemplo, o programa, ao ser executado, gera um processo que pode demorar, impedindo a execução de outros processos
- Exemplo (com thread):
```c
long int soma[2];

void somaParte1() 
    int i;
    long acc = 0;

    for (int i = 1; i <= 5000000; i++) {
        acc += i;
    }
    soma[0] = acc;
}

void somaParte2() {
    int i;
    long acc = 0;

    for (int i = 5000001; i <= 10000000; i++) {
        acc += i;
    }
    soma[1] = acc;
}

int main() {
    int t1, t2;
    
    pthread_create(&t1, NULL, somaParte1(), NULL);
    pthread_create(&t2, NULL, somaParte2(), NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    printf("A soma é %ld\n", soma[0] + soma[1]);

    return 0;
}
```

- Exemplo 2 (com thread):
```c
long int soma[4];

void somaParte(int parte) {
    int i, inicio, fim;
    long acc = 0;

    soma[parte] = 0;

    inicio = 1 + parte * 2500000;
    fim = (1 + parte) * 2500000;

    for (i = inicio; i <= fim; i++) {
        acc += i;
    }

    soma[parte] = acc;
}

int main() {
    int t1, t2, t3, t4;

    pthread_create(&t1, NULL, somaParte, 0);
    pthread_create(&t2, NULL, somaParte, 1);
    pthread_create(&t3, NULL, somaParte, 2);
    pthread_create(&t4, NULL, somaParte, 3);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);
    pthread_join(t4, NULL);

    printf("A soma é %ld\n", soma[0] + soma[1] + soma[2] + soma[3]);

    return 0;
}
```
- Caso o array global fosse utilizado durante o loop da soma, o cache L1 seria invalidado para as outras CPUs 2500000 vezes. Por isso, a criação de uma variável local de acúmulo faz a invalidação de cache só ocorrer uma vez por thread. De qualquer forma, como o uso do array é feito em posições de memória diferente, isso não gera problema.
- Cada criação de thread cria uma pilha nova. Por isso, o número de threads deve ser coerente com o número de CPUs e com a variação do tempo conforme o aumento de threads
- CURIOSIDADE: O hyper-threading da Intel consiste em um único core simula 2, gerando a troca de contexto em hardware (pois normalmente é em SO, com o processo de restauração de registrador). Isso é feito com uma troca de registradores, sem precisar preservar em memória
