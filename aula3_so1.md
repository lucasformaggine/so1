# Sistemas Operacionais I

## Aula 3

### Resumo da última aula
- O sistema operacional é um software que promove o gerenciamento e compartilhamento dos recursos do
hardware para os processos
- SO fornece serviços adicionais ao hardware (máquina estendida)
- Tipos de serviço oferecidos:
    - Execução de programas
    - Acesso a dispositivos
    - Acesso a arquivos
    - Acesso a interface com o usuário
    - Tratamento de dados
    - Construção de uso e de recursos
- SO como administrador de recursos
    - Controle de acesso ao recurso
    - Compartilhamento do recurso

### A estrutura interna dos sistemas operacionais
- O SO é um software que é dividido internamente
- Tipos de núcleo
    - Monolítico
        - Não há uma divisão clara das divisões internas;
    - Em camadas
        - Há uma divisão interna
        - "Device Drivers": módulos do SO que fazem comunicação com o dispositivo, como impressora, disco magnético, placa de rede, disco rígido)
        - Camadas de rede
    - Microkernel
        - Partes do SO precisam fazer parte do núcleo, embora haja algumas externas (ex: gerenciamento de arquivos, interface gráfica do usuário do Linux, embora este não seja microkernel)
        - No caso divisão de memória, ela até poderia ser feita fora do núcleo, embora não seja
            - Vantagens: diminuiria a dependência tornando o sistema mais modularizado, facilitando versionamento
            - Desvantagens: perda de desempenho (mudança no modo de operação entre operações internas e operações externas, já que cada chamada gera um gasto)
        - O Unix, embora peque pela perda de desempenho, facilita operações sobre interfaces do usuário; isso não ocorreu no Windows, o que dificultou features para tal funcionalidade
        - Em mainframes, a impressão de arquivops ocorria dentro do kernel
        - Serviços como de impressão exemplificam o ato de dar responsabilidade para algo fora do núcleo, assim como ocorre em vários serviços em SO microkernel
### Separações dos SOs na prática:
- SOs de máquinas servidoras
    - Em maqúinas servidoras, como o acesso ao servidor é feito em arquitetura de microserviços, são privilegiados os desempenhos para operações justamente para atender outras máquinas, o que implica em uso mais fraco de interface com o usuário
- SOs de máquinas clientes
- SOs de celulares/tablets
    - Foco em economia de energia: quando não há uso, ocorre o desligamento da CPU
- SOs embarcados
    - Realizam uma função específica. Ex: roteador, máquinas fotográficas
- SOs máquinas simples
    - Utilizam sensores
- SOs Real-Time
    - Possuem rotinas de processamento bastante curtas, executando as tarefas no menor tempo possível
    - Forte paralelismo e controle de priorização das atividades
    - Foco no escalonamento de processos
    - Ex: serviços de aviação, serviços médicos de batimentos cardíacos etc.

### Revisão da organização de computadores

- Processador (CPU)
- Memória
- Registradores
    - De uso geral
    - De propósito específico
        - Program Counter (PC): indica a posição atual na sequência de execução do processo; armazena o endereço da instrução sendo executada ou da próxima execução; só é acessado de forma indireta
        - Control/Status Register: guarda informações "flag", como carry, overflow e zero, para tomar decisões if-then e cálculos bit-a-bit; só é acessado de forma indireta
        - Stack Pointer (SP): guarda o topo da pilha; tem acessos diretos e indiretos (no retorno de subrotinas)
- O SO tem que conhecer os registradores, pois eles precisam ser preservados na troca de contexto e nas interrupções de CPU.


```c
#include <stdio.h>

int main() {
    int i = 0;

    i = i+5;

    return 0;
}
```

```asm
mov eax, [5000]
add eax, 5
mov [5000], eax ;Interrupção
```

- Durante a interrupção, o SO chama uma rotina tratadora de interrupção. Ex:

```asm
    in eax, 90 ;Recebe a tecla do teclado
    mov [80000], eax ;Endereço de memória do sistema operacional
    iret ;Fim do tratamento da interrupção
```
- Isso está incorreto!
- Caso seja utilizado o mesmo registrador, haverá perda e portanto inconsistência nos dados caso haja subrotinas sendo executadas em paralelo (ex: digitar no teclado enquanto roda um código)
- Ao invés, os dados devem ser guardados em uma pilha durante a interrupção e só retornar quando a interrupção tiver sido finalizada


### Barramento de memória

- A conexão entre a CPU e a memória é feita por um conjunto de fios condutores chamado de barramento de memória
- A CPU para quando faz a requisuição a memória, o que gera custo de alguns ciclos da CPU
- Existe uma memória cache intermediária entre a CPU e a memória. Ela é mais rápida do que a memória principal, porém é cara, portanto armazena menos bytes
- A memória cache se preocupa em guardas bytes que foram recentemente trazidos da memória principal; isto é, o valor fica na memória mas também na memória cache. Caso a CPU precise consultar o mesmo endereço, como o valor já está na memória cache, essa consulta é feita mais rapidamente
- No caso de escrita, o valor é alterado em ambas as memórias, vindo novamente da memória cache
- Mesmo com uma boa velocidade, apenas uma memória cache não seria suficiente para facilitar o uso rápido de acesso aos dados em memória, o que exige a hierarquização de níveis de memória cache:
    - L1: mais rápida, menos armazenamento, mais próxima da CPU
    - L2:
    - L3: menos rápida, mais armazenamento, mais afastada da CPU
- Caso demore para consultar um dado, a L1 pode ficar cheia e então o dado é consultado em L2, um pouco mais devagar. Caso demore mais ainda, então o L2 também deve estar cheio, logo o dado deve estar em cache nível L3. Em último caso, caso todos os caches estejam cheios, então o dado é consultado na memória principal

### Chip Multicore

- Imagine um processador com 4 cores
- Para cada core, tem um cache L1
- O cache L2 é compartilhado
- Uma alteração no L1 de um core altera o cache compartilhado L2 e a memória compartilhada, mas em teoria não altera os demais L1s
- Se os outros cores perceberem que há uma alteração de uma variável presente no cache, o cache precisa ser limpo, o que levará a uma perda de desempenho
- Os SOs precisam evitar que a mesma variável seja usada por CPUs diferentes por conta da invalidação por cache L2 (ou por cache L3, caso haja mais chips ou mais CPUs)


### Interrupções
- Eventos que desviam o fluxo de execução da CPU
- A CPU para temporariamente de executar o que estava
sendo executado e executa uma rotina tratadora do tipo de interrupção
- Quando a rotina tratadora acaba, a instrução que estava executando é reiniciada (para evitar inconsistência de uso da memória)
- Para cada tipo de interrupção, existe uma rotina tratadora diferente
- Existem números para representar o tipo de interrupção

### Tipos de Interrupção
- Gerada por dispositivo
    - Existem 2 lógicas: quando recebe um dado do dispositivo (a interrupção ocorre quando o dispositivo tem um dado novo para ser lido, no caso de teclados quando alguém digita uma tecla, no caso de placa de rede quando recebe um pacote de rede); quando envia um dado do dispositivo (ocorre uma interrupção quando o dispositivo é capaz de receber um dado novo; no caso do dispositivo não estar fazendo nada a interrupção é instantânea)
    - Exemplos:
        - Disco magnético
        - Placa de rede
        - Teclado
        - Relógio (interrupção após um determinado intervalo de tempo através de um circuito que simula um relógio)
- Gerada pelo próprio programa
    - Trap
- Situações de erro
    - Divisão por zero
    - Acesso indevido à memória
    - Interrupção de software
        - Instrução em linguagem de máquina que gera uma interrupção
- Interrupções simultâneas
    - Durante a rotina de uma interrupção, ocorre outra interrupção
    - Nesse caso, é necessário priorizar as interrupções de acordo com uma hierarquia pelo tipo de interrupção
    - A hierarquização é feita pelo SO, enquanto a interrupção por conta da priorização é feita pelo hardware

### Chamadas ao SO

- O SO não é um programa convencional composto por rotinas em que uma chama a outra, mas sim há vários pontos de entrada que rodam rotinas diferentes
- Funcionam como um servidor remoto




### Desafio do Paulinho
http://acmph.blogspot.com/2018/06/uva-10688-poor-giant.html

```cpp
#include <iostream>
#include <vector>
#include <climits>

using namespace std;

int main() {
    int t;
    int n, k;
    int applesInInterval;

    cin >> t;

    for (int caseCounter = 1; caseCounter <= t; caseCounter++) {
        cin >> n >> k;

        vector<vector<int>> dp(501, vector<int>(501));


        for (int i = n; i >= 1; i--) {
            for (int j = i; j <= n; j++) {
                dp[i][j] = INT_MAX;
                if (i == j) {
                    dp[i][j] = 0;
                    continue;
                }

                for (int d = i; d <= j; d++) {
                    applesInInterval = j - i + 1;
                    dp[i][j] = min(dp[i][j], dp[i][d-1] + dp[d+1][j] + applesInInterval * (k + d));
                }
            }
        }

        cout << "Case " << caseCounter << ": " << dp[1][n] << endl;
    }
    return 0;
}
```
