# Aula 9 - SO1
- Sistemas Operacionais Modernos - Tanerbaum

## Monitor

- É um módulo de uma linguagem, onde cada rotina tem exclusão mútua
- Somente 1 thread dentro do monitor.
- Se 1 thread chama 1 rotina do monitor, as outras ficarão bloqueadas ao acessar qualquer outra rotina do monitor.
- O monitor tem um semáforo mutex de exclusão mútua embutido.
- Rotinas dentro do monitor que executam com exclusão mútua
    - Exemplo real: synchronized em Java.
- O monitor tem variáveis de condição e rotinas para usar com estas.
- Evita erros de programação, automatizando a implementação de trechos críticos (existe uma exclusão mútua implícita nas rotinas do monitor)
- O monitor não trata controle de recursos, apenas controle de trechos críticos. Portanto, não se deve utilizar semáforos quando se utiliza monitores
- Variáveis usadas pelo monitor não podem ser usadas por código fora do monitor, pois assim garantem que essas variáveis possam ser manipuladas sem problemas com o trecho crítico pelas rotinas que essas a utilizam
- Existe o conceito de variável de condição que controla o uso de recursos dentro do mionitor
- Existem 2 rotinas para usar as variáveis de condição: uma se chama "wait" e a outra se chama "signal".
- A rotina "wait(c)" libera atomicamente o trecho crítico e bloqueia a thread que chamou. Lembra a rotina "down(s)"
- A rotina "signal(c)" libera uma thread bloqueada (na variável de condição) para ser executada. Lembra a rotina "up(s)"
- Pode ainda haver a rotina "broadcast(c)", que libera para execução todas as threads bloqueadas
- Exemplo em C (hipotético, pois o "monitor" não existe de forma nativa em C; embora exista em Pascal e Java(synchronized -> exclusão mútua))

```c
const int TAM_MAX = 5;

Monitor ProdutorConsumidor {
    int in = 0;
    int out = 0;
    int qtd = 0;

    int a[TAM_MAX];

    VariavelDeCondicao vazio;
    VariavelDeCondicao cheio;

    int get() {
        int item;

        if (qtd == 0) {
            wait(vazio);
        }

        qtd = qtd - 1;
        item = a[out];
        out = (out+1) % TAM_MAX;

        if (qtd == TAM_MAX-1) {
            signal(cheio);
        }

        return item;
    }

    void put(int item) {
        if (qtd == TAM_MAX) {
            wait(cheio);
        }

        qtd = qtd+1;
        a[in] = item;
        in = (in+1) % TAM_MAX;

        if (qtd == 1) {
            signal(vazio);
        }
    } 
}
```

- Exemplo com 1 consumidora e 1 produtora (eixo y é o tempo)

T1 (consumidora)                                T2 (produtora)

// qtd é 0

if (qtd == 0) { 
    wait(vazio); -------> troca contexto        if (qtd == TAM_MAX) {
                                                    wait(cheio);
    }                                           }
    // bloqueada                                qtd = qtd+1;
    // qtd vai pra 1                            a[in] = item;
    // bloqueada                                in = (in + 1) % TAM_MAX;
    // bloqueada                                if (qtd == 1) {
    // liberada       <---troca contexto           signal(vazio);
    // ...                                      }
    } 
}


- Exemplo com 2 consumidoras e 1 produtora

T1 (consumidora)                                T2 (produtora)                      T3 (consumidora)

// qtd é 0

if (qtd == 0) { 
    wait(vazio); -------> troca contexto        if (qtd == TAM_MAX) {            
}                                                   wait(cheio);
                                                }
    // bloqueada                                qtd = qtd+1;
    // qtd vai pra 1                            a[in] = item;
    // bloqueada                                in = (in + 1) % TAM_MAX;
    // bloqueada                                if (qtd == 1) {
    // bloqueada                                    signal(vazio);
    // pronta                                   } ------------> troca contexto      if (qtd == 0) {
    // ...                                                                              wait(vazio);
    }
                                                                                    qtd = qtd - 1; // qtd é zero
                                                                                    item = a[out];
                                                                                    out = (out + 1) % TAM_MAX;
                                                                                    if (qtd == TAM_MAX-1) {
                                                                                        signal(cheio)
                                                                                    }
                                                                                    return item;
                                                    <-------- troca contexto
    qtd = qtd-1; // qtd vai dar negativo   
}

- Exemplo com 2 produtores e 1 consumidor é análogo, porém a quantidade estouraria o tamanho máximo
(isto é, quando um produtor incrementasse a qtd e o contexto fosse para o outro produtor que estava no wait(cheio), ele aumentaria a qtd)

- Solução 1: monitor original de Hoare
- Garante que a troca de contexto será com a thread específica que acabou de sair de um wait (cheio ou vazio), evitando que valor seja decrementado ou incrementado mais do que o necessário
- Isto é, a thread que recebeu o sinal (ex: T1) deve ser executada imediatamente e bloquear a thread que gerou o sinal (ex: T2). Acaba sendo um comportamento intuitivo do monitor, porém é mais complicado de implementar, pois exige que o SO tenha um mecanismo de determinar qual deve ser a thread a ser executada
- Solução 2: trocar "if" por "while" em alguns pontos

No get(): "while(qtd == 0)"
No put(): "while(qtd == TAM_MAX)"

- Sendo assim, no segundo exemplo, ele pode rodar T2, depois T3, e ao voltar para T1, ele voltará para o while; ao verificar que a qtd é zero, ele roda o wait novamente.

### Diferenças entre o semáforo e a variável de condição
- Semáforo guarda informações passadas; a variável de condição não guarda
- As operações de up e down são comutativas; o wait e o signal não, caso contrário a última thread a fazer wait ficará esperando indefinidamente. Isto é, enquanto no primeiro caso o resultado final é o mesmo, no segundo caso não é.
- Wait em variável de condição sempre bloqueia a thread; o down em semáforo não (ele só bloqueia quando o valor do semáforo é zero)
- Variáveis de condição precisam estar dentro de uma região crítica


### Troca de mensagem

- É o envio de um conteúdo entre processos
- Um processo pode fazer send ou receive
- O método "send(identificador, mensagem)" envia o conteúdo, enquanto o método
"receive(mensagem)" recebe o conteúdo
- Caso um processo faça receive antes de outro processo fazer send, o primeiro processo é bloqueado
- Quando há armazenamento, como pode haver mais de uma mensagem sendo enviada ao mesmo tempo, o SO precisa armazenar mais de uma mensagem através de um enfileiramento. Sendo possível, ele implementa um "ProdutorConsumidor" para gerenciar esse enfileiramento
- Quando não há armazenamento, se o processo 2 faz send antes de outro processo fazer receive, o processo 2 é bloqueado, já que não há para onde a mensagem ir se ela não pode ser armazenada; é necessário que haja alguém esperando para receber para que seja feita a cópia da mensagem para o outro processo
- O "send" possui dois identificadores: um identificador de processo e um identificador de caixa postal
- Pode existir ou não uma unidade de comunicação. Caso exista, o recebimento e o envio da mensagem é sempre da ordem da mensagem (ou seja, a mensagem inteira é enviada e recebida). Caso não exista, o envio é de bytes e o recebimento é de bytes.
- Antes de se usar essa ideia com arquivos, a troca de mensagens ocorria com pipes, de modo que a mensagem saía da entrada padrão e ia para a saída padrão. Futuramente, as pipes se tornaram "pipes nomeados".
- No Unix, o mais comum é usar chamadas ao SO que não têm conceito de unidade:
    - Pipes: processos que se comunicam precisam ser parentes, pois já que não há nomeação, não há como identificar a pipe a não ser pelos seus pais e filhos; unidirecionais
    - Pipes nomeadas: processos que não precisam ser parentes, mas precisam saber o nome da pipe; tipicamente unidirecionais, mas podem ser bidirecionais
    - Sockets: é uma estrutura que permite processos em diferentes dispositivos e de forma bidirecional, abstraindo uma camada de redes
- No Macintosh, troca de mensagens entre processos tipicamente não faz cópia da mensagem, e sim um "copy on write": ao mandar copiar uma mensagem, ela não é propriamente copiada, apenas sinalizada, podendo ter trechos utilizados caso necessário; porém, caso haja alguma alteração no destino da mensagem, então de fato a cópia é feita, mas apenas do trecho onde há alteração. Isso resolve o problema do tempo gasto com a cópia de mensagens e o próprio gasto de memórias. OBS: esse assunto só será aprofundado em SO2
- **FINAL DA MATÉRIA DA PROVA**

 
