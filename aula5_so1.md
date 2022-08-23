# Resumo da última aula
## Modo de operação
- Usuário (não privilegiado)
- Núcleo (privilegiado)
    - controlar algum dispositivo
    - controlar memória

## Relação programa x processo
Execução de diferentes processos ao longo do tempo com diferentes situações de processadores
```
progama = receita  
cpu = cozinheiro  
processo = ato de fazer o bolo
```

## Execução e controle de processos pelo usuário
    - Execução em paralelo
    - Entrada, saída e erro padrão
    - Encadeamento de processos


# Aula 13/07
## Chamadas ao SO para lidar com processos
- Criar um processo (mais difícil)
- Terminar um processo
    - Auto-terminar o processo
    - "Matar" um processo qualquer

## Terminar um processo
### Todo processo tem uma rotina principal que é executada primeiro.
Por exemplo, em C, a rotina principal é a main() e ao utilizar o int main(), esse inteiro é uma indicação se o processo terminou bem ou mal.  
- Se terminou com 0, terminou bem
- Se diferente de 0, não

**Exemplo 1**  
bash script que retorna a saída de um comando no meio do arquivo
``` bash
cp file1 file2
echo $*
```

**Exemplo 2**  
``` c
int main() {
    return 3;
}
```

Nesse caso, não há nenhuma chamada ao SO, apenas um programa que retorna um código de erro para quem chamou esse programa.

**Exemplo 3**  
```c
int x() { 
    ...
    if (percebi_um_erro) { 
        return -1;
    } 
    ...
    return 0;
}

int main() {
    ...
    f = x();
    if (f < 0) {
        return 3;
    }
    ...
    return 0;
}
```
Nesse caso podemos ver que a falha ocorreu em uma rotina x, que desencadeou um erro na rotina principal.

**Exemplo 4**  
```c
int x() {
    if (percebi_um_erro) {
        exit(3); // chamada ao SO para terminar o processo
    }
}
int main() {
    ...
    j = x();
    ...
    return 0;
}
```
- No windows, ExitProcess(codigo_erro);

**Exemplo 5**  
``` bash
sort ... & pid = 58
kill 58
```
"Mata" o processo de PID 58  
PID: Process Identifier (identifica o processo de forma única)

### Sinal
A ideia é parecida com as interrupções, mas não é igual.  
- Interrompe a qualquer momento e enviada pelo hardware
- O sinal pode ou não ser enviada pelo hardware, mas também pode ser enviada por um comando
    - Por exemplo, o comando kill
    - Para o que está fazendo e trata o sinal
    - Um processo que manda um sinal para outro
Se no meio da rotina, o processo recebe um sinal, ele para a rotina para lidar com o sinal

**Exemplo**  
```
void Y() {
    ...
    read()
    ...
}

int rotina_tratadora_de_sinal() {
    ...
}
```
Se no meio da rotina Y receber um sinal, ele para a rotina e vai para a rotina_tratadora_de_sinal.

Existem diversos tipos de sinais. Por exemplo, o sinal que mata um processo é enviado pelo comando kill.  

**Possíveis comportamentos de um processo quando ocorre um sinal**  
- Ter uma rotina tratadora de um sinal (exemplo)
- Matar o processo (opção default para a maioria dos sinais)
- Ignorar o sinal (em alguns poucos casos, o default é ignorar o sinal)

**Exemplo**  
Ao apertar o ctrl+C em algum programa executando no terminal, o SO envia um sinal para o processo que está executando para terminar o processo.

**Exemplo**  
Imagine que um programador crie uma rotina para ignorar todos os sinais e dessa forma, não seria possível terminar o programa. Nesse caso, existem dois sinais que não podem ter o comportamento alterado
- SIGKILL 
- ?
```c
void Y() {
    ...
    signal(7, rotina_tratadora_de_sinais); // Muda o comportamento do processo para o sinal 7
    read();
    ...
}

int rotina_tratadora_de_sinais() {
    ...
}
```

**Exemplo de sinal enviado pelo SO enviado pelo hardware**  
Caso o processo tente acessar a memória, o hardware **envia** uma interrupção e o SO **trata** essa interrupção e envia um sinal para o processo, que pode "pedir" para a rotina não ser abortada.

**Exemplo windows**  
``` bat
linha_de_comando
taskkill 58
taskkill 58 /f
```

Se utilizar o /f, uma possível rotina de finalização não vai ser seguida, então geralmente enviamos o taskkill sem o force, e no caso de não finalização, enviamos com /f.

```
TerminateProcess(PROCESS_HANDLE, codigo_sinal);
```
PROCESS_HANDLE -> Não é o PID, tem que fazer uma conversão  
É possível alterar o código de retorno de terminar um processo.

## Criar um processo
### CreateProcess, Parâmetros:
- Caminho / nome do arquivo executável
- Parâmetros que chegam ao processo
- Endereço do "ambiente" do processo
- Diretório onde o novo processo vai executar (diretório corrente do novo processo)
- Prioridade do novo processo

### Variáveis de ambiente
São as variáveis mais estáveis e que não mudam tanto

**Exemplo**  
- LANGUAGE - Diz qual a lingua que você está

**CreateProcessAsUser**: cria um processo com um usuário específico
- Pode ser que o processo não tenha permissão para fazer algo, então podemos alterar o dono do processo
- Importante pois as permissões estão ligadas ao usúario que enviou o processo.
- Mesmos parâmetros da original mais o parâmetro do usúario

**No unix, a chamada de criação tem 0 parâmetros.**
- Como é possível começar um programa onde não temos nenhum parâmetro? 

PROCESSO X (ORIGINAL)
```c
int main () {
    int i, j;
    j = 3;
    i = fork(); // cria um processo novo que é uma cópia do processo X e começa a partir dessa linha
    if (i != 0) {
        j = 5;
        printf("Sou o pai. Meu filho é %d, j = %d", i, j);
    }
    else {
        printf("Sou o filho. j = %d", j);
    }
}
```
PROCESSO Y (NOVO)
```c
int main() {
    int i, j;
    j = 3;
    i = fork(); // começa a partir daqui com i = 0
    if (i != 0) {
        j = 5;
        printf("Sou o pai. Meu filho é %d, j = %d", i, j);
    }
    else {
        printf("Sou o filho. j = %d", j);
    }
}
```

PROCESSO ORIGINAL: retorno do fork é o PID do processo novo  
PROCESSO NOVO: retorno do fork é ZERO

**Exemplo 2**
PROCESSO X (ORIGINAL)
```c
int main () {
    int i, j;
    j = 3;
    i = fork(); // cria um processo novo que é uma cópia do processo X e começa a partir dessa linha
    if (i != 0) {
        printf("Sou o pai.");
    }
    else {
        // faço alguma coisa no processo do filho
        execve("word", parametro, end_ambiente);
        printf("Depois do execve");
    }
}
```
PROCESSO Y (NOVO -> DESCARTADO)
```c
int main() {
    int i, j;
    j = 3;
    i = fork(); // começa a partir daqui com i = 0
    if (i != 0) {
        printf("Sou o pai.");
    }
    else {
        // faço alguma coisa no processo do filho
        execve("word", parametro, end_ambiente);
        printf("Depois do execve");
    }
}
```
PROCESSO Y (DESCARTA DA MEMÓRIA O ANTIGO PROCESSO Y E CRIA UM NOVO PROCESSO COM O MESMO PID)
```
CÓDIGO DO WORD
```

Nesse caso, temos dois passos:
1. Primeiro cria-se um PROCESSO Y
2. Dentro do PROCESSO Y, entramos na condição do if que criamos um novo PROCESSO Y com o mesmo PID e descartamos o atual da memória. 

Dessa forma, é possível alterar a forma como o word vai funcionar, já que temos código antes do execv. Por exemplo:
- alterar o diretório do arquivo que o word vai abrir,
- alterar parâmetros e variáveis

Qualquer linha após o execv não será executada. Então, o `printf("Depois do execve");` não será executado no código do exemplo.

Há apenas um caso onde as linhas serão executadas e é quando execve dá erro.
- Por exemplo, se o word não achar o arquivo para abrir, o programa continua

**DESVANTAGEM** - Gasta-se tempo copiando o processo pai todo do processo original, o que não há no windows. Porém, não é assim que funciona, no unix temos o sistema copy on write onde a cópia só ocorre de fato quando um dos lados é alterado.

## Relacionamento entre programa x processo
- NÃO UNIX  
```
PROGRAMA 1 --- n PROCESSO
```

- UNIX
```
PROGRAMA n --- n PROCESSO
```
Pode ter vários programas do mesmo processo em diferentes intervalos de tempo, como é o caso do exemplo acima onde temos 2 programas (c e word) para o mesmo PROCESSO Y.

Os programas rodam em paralelo, mas existem casos que gostaríamos de rodar os programas em paralelo. 

**exemplo**
PROCESSO X (ORIGINAL)
```c
int main () {
    int i, j, status_filho;
    j = 3;
    i = fork(); 
    if (i != 0) {
        printf("Sou o pai.");
        j = wait(&status_filho); // Aqui o programa espera o novo processo criado acabar e retorna o número do processo que terminou
    }
    else {
        execve("word", parametro, end_ambiente);
    }
}
```
PROCESSO Y (NOVO -> DESCARTADO)
```c
int main() {
    int i, j;
    j = 3;
    i = fork(); // começa a partir daqui com i = 0
    if (i != 0) {
        printf("Sou o pai.");
    }
    else {
        // faço alguma coisa no processo do filho
        execve("word", parametro, end_ambiente);
        printf("Depois do execve");
    }
}
```
PROCESSO Y (DESCARTA DA MEMÓRIA O ANTIGO PROCESSO Y E CRIA UM NOVO PROCESSO COM O MESMO PID)
```
CÓDIGO DO WORD
```

**Curiosidade 1** Quando o processo filho acaba antes do pai fazer o wait, como o código do filho deve ser passado para o pai, isso faz com que o filho se torne um processo zumbi e guarda no SO o código a ser passado para o pai.

**Curiosidade 2** Quando o processo pai acaba antes do filho sem fazer o wait, então o filho passa a ser filho do processo init. 

**Curiosidade 3** O processo 1 do unix é o init e ele vai gerando diversos filho.