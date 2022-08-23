# Aula 10 - Sistemas Operacionais I

## Resumo da última aula

- Monitores: exclusão mútua implícita
- Variáveis de condição
- Variações de monitores:
    - Hoare: execução passa diretamente ao processo que recebeu o sinal
    - Sem regra de ordem de execução (Java) - necessário alterar o código do processo que recebe o sinal
- Troca de mensagens
    - Identificação destinatário
        - Processo
        - Caixa postal
    - Unidade de transmissão (sim ou não)
    - Armazenamento da mensagem (sim ou não)
    - Crítica à troca de mensagens: duplicação do gasto de memória; tempo gasto para a cópia
    - Solução: copy on write (Mach)

## Deadlock

- "Problema dos filósofos" exemplifica o deadlock
- Exemplo em código:

```c
// semáforo a = 1 e semáforo b = 1
// Thread 1                // Thread 2
down(A);                   down(B);
// codigo da thread 1      // codigo da thread 2
down(B);                   down(A);
// codigo da thread 1      // codigo da thread 2
up(B);                     up(A);
up(A);                     up(B);
```

- No exemplo acima, embora ambos cnsigam fazer o primeiro down, o segundo down é impossibilitado pelo semáforo estar zerado; como ambos ficam no down aguardando o up do outro, ocorre um deadlock
- Deadlock ocorre quando:
    - Processos(threads) competem por acesso a recursos limitados
    - A sincronização de processos não está programada corretamente
- Definição: um deadlock existe em um conjunto de processos/threads quando cada processo/thread está esperando por um evento que apenas pode ser gerado por outro processo/thread no conjunto
- Suponha a seguinte situação: 
    - (Px) <- [Ry]: um processo X está utilizando um recurso Y (ex: filósofo 0 pegou o garfo 0)
    - (Pw) -> [Rz]: um processo W está esperando um recurso Z (ex: o filósofo 0 está esperando o garfo 1 ser liberado)
    - *foto 10-1*
    - Outro exemplo: carros "presos" no trânsito em um cruzamento
    - *foto 10-2*
- Quatro condições para ocorrer o deadlock
    1) Exclusão mútua no acesso ao recurso
    2) Hold and wait: um processo está acessando um recurso e está esperando para acessar outro recurso que está sendo acessado por um outro processo
    3) Não preempção do recurso: o recurso só é liberado voluntariamente pelo processo
    4) Espera circular
- Quatro abordagens para o deadlock:
    1)  Não fazer nada de forma automática, supondo que a possibiliade de ocorrência seja baixa. Caso pareça que haja deadlock, se faz uma intervenção manual
    2) Prevenção: eliminar alguma das 4 condições necessárias ao deadlock
    2.1) Solução possível: amostras de recurso, porém de forma virtualizada. O processo tiliza um recurso virtualizado achando que está usando o recurso verdadeiro. Ex: spooler de impressão
    2.2) Eliminar o hold and wait. Solução: um processo só pode pedir recursos quando não está utilizando nenhum e precisa pegar os recursos todos de uma vez. Isso não ocorrerá em todos os SOs; nesse caso, é necessário implementar uma primitiva que "teste o down" para ver se haverá o bloqueio ou não para não fazer um down que possa gerar deadlock, testando de tempo em tempo a possibilidade de fazê-lo. Crítica: pode ser ineficiente no uso dos recursos: um recurso alocado a um processo pode gastar um tempo razoável sem ser utilizado
    2.3) Eliminar a não preempção do recurso: só pode ser usado se existir um mecanismo de preservação/restauração do estado do recurso. Ex: a impressora não poderia "parar" uma impressão, realizar outra e retornar para a primeira
    2.4) Eliminar a espera circular. Solução: atribuo um número a cada tipo de recurso. Cada processo só pode acessar os recursos na ordem numérica. Pensando em uma estrutura "circular", o último processo (que fecharia o ciclo) não pegaria os recursos no mesmo sentido; a inversão de ordem impede o deadlock.
    3) Evitar deadlocks: controle de concessão do recurso. O SO apenas deve permitir a alocação quando esta for segura.
        - Estado seguro: existe uma sequência em que todos os processos terminam sem que ocorra deadlock
        - Para que o SO implemente o algoritmo de concessão de recursos, o processo informa previamente a quantidade total de recursos de cada tipo que ele vai precisar
        - Exemplo: um computador tem 12 unidades de fita.

        Processos|Necessidade máxima|Uso atual|
        ---------|------------------|---------|
        P0       |10                |5        |
        P1       |4                 |2        |
        P2       |9                 |2        |

        - Unidades de fitas livres: 3

        - P1 pode receber as 2 unidades de fita restantes, por ser menor ou igual à quantidade de unidades de fita livres. Dessa forma, ele consegue satisfazer sua necessidade máxima de 4, liberando-as e deixando 5 unidades de fita livres.
        - P0 pode receber as 5 unidades de fita por ser menor ou igual ao número de unidades de fitas livres. Dessa forma, ele consegue satisfazer sua necessidade máxima de 5, liberando-as e deixando 10 unidades de fita livres.
        - Agora, P2 pode receber 7 unidades de fita livres por ser menor ou igual ao número de unidades de fita disponíveis. Então, ele satisfazer sua necessidade. Ao final, ficam as 12 unidades de fita disponíveis e todos os processos usaram o que precisavam.
        - Conclusão: o estado original é seguro, dando unidades de fita para o P1.

        - A partir da situação original, P2 pede mais uma unidade. O SO simula a concessão desta nova unidade

        Processos|Necessidade máxima|Uso atual|
        ---------|------------------|---------|
        P0       |10                |5        |
        P1       |4                 |2        |
        P2       |9                 |3        |

        - Unidades de fitas livres: 2

        - Nesse caso, o P1 é o único que consegue utilizar unidades de fita. Pegando as 2 livres, ele consegue satisfazer sua necsesidade, liberando as 4 unidades de fita em uso. A quantidade de unidades de fita livres é 4.

        - Agora, o P0 não consegue pedir 5 pois só há 4 unidades livres. Nem o P2 consegue pedir pois não satisfaria, já que ele precisa de 6 e só tem 4 disponíveis. Então, nessa situação, o SO não dá dessa unidade de fita para o P0. 

        - Para cada "pedido de unidade de fita livre" de um processo, o SO "roda o algoritmo" para testar se isso gerará garantidamente um estado seguro. Se houver possibilidade de deadlock, o recurso não é cedido.

        - Algoritmo generalizado (para mais de um tipo de recurso) se chama algoritmo do banqueiro:
            - Cliente do banco (processo) -> cliente de crédito (uso máximo de recurso)
            - O banqueiro (SO) verifica se cada empréstimo pode ser concedido ou não

    4) Detecção e resolução de deadlock: de forma automática, o SO de tempos em tempos executa o algoritmo de detecção de deadlock; caso exista um deadlock, o SO tenta resolver, por exemplo, terminando um dos processos. Como existe o risco desse processo ter salvo coisas durante seu processamento, este pode estar inconsistente pela incompletude após seu término, é necessário setar checkpoints que salvem os dados para uma possível restauração, o que costuma ocorrer em SGBDs (transaction -> atomicidade (ou tudo é feito, ou nada é feito))
        - Exemplo: *foto 10-3*
        - Um recurso que não esteja bloqueado pode liberar sua instância e evitar que o "grafo dos processos e recursos" gere um ciclo, logo não haverá deadlock. Se isso não for possível, então haverá um ciclo (juntamente com as outras 3 condições), logo haverá deadlock
        - Algortmo de detecção de deadlock
            - "disponivel": vetor com um elemento para cada tipo de recurso
            - "alocacao_atual": matriz, na qual as linhas são os processos e as colunas são os tipos de recurso
            - "work": vetor temporário com um elemento para cada tipo de recurso
            - 
            - Pseudocódigo:
                ```c
                    int n;
                    int disponivel[n];
                    int alocacao_atual[n][n];
                    int work[n];
                    
                    work = disponivel();
                    while (1):
                        encontre_processo_que_nao_tenha_terminado();


                ```

    



    
    
    