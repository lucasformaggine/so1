# Sistemas Operacionais I

## Aula 4

### Resumo da última aula
- Organização de computadores
    - CPU
    - Memória
    - Registradores
- Processadores multinúcleo (multicore)
    - Problema da invalidação da cache nivel 1
- Dispositivos
    - Armazenam dados; interface humana; comunicação
- Interrupções
    - Tipos:
        - Geradas pela ação do programa (na verdade, é gerada pela CPU)
        - Geradas pelo dispositivo
        - Acionadas explicitamente pelo programa (após uma instrução em linguagem de máquina)
    - Interrupções simultâneas: existem prioridades

### Modo de Operação
- É razoável diferenciar o que um programador comum pode fazer com relação a um desenvolvedor de um SO
- Se o programa der uma ordem errada à placa de vídeo, pode gerar problemas que façam a imagem sumir
- É necessário um sistema de privilégios para evitar acessos indevidos; isto é, um programa normal não poder alterar variáveis do sistema operacional
- É necessário restrições para os endereços memórias de um programa normal, enquanto um SO não possui restrições
- Diferentes modos de operação:
    - Usuário (não privilegiado)
    - Núcleo/sistema (privilegiado)
- Restrições do modo não privilegiado:
    - Ao executar instruções que façam comunicação direta com o dispositivo (é gerada uma rotina de interrupção do SO que aborta a execução da instrução)
        - No uso de uma máquina virtual, a interrupção é seguida do uso da máquina virtual, logo as instruções que são restrigindas na máquina real são simuladas/emuladas na VM. Isso também ocorre em jogos de MSDos que rodam Windows
    - Ao acessar uma parte da memória não permitida
        - O efeito é uma interrupção gerada pelo hardware através de um chip da CPU chamado Memory Management Unit (MMU), que controla o acesso a memória e também gera interrupções
        - A interrupção é feita na rotina de tratamento adequada; tipicamente, o SO aberta o processo
        - Para que haja uma chamada ao SO, é necessário que o modo de operação seja privilegiado. No caso de acesso indevido à memória, isso é chamado pela MMU
### Programa vs Processo
- Programa executável também é chamado de arquivo. Ele fica em disco.
- Programa em execução também é chamado de processo. Ele fica em memória: 

- Analogia:
    - O cozinheiro usa ingredientes, segundo uma receita, para fazer um bolo
    - Cozinheiro: CPU
    - Ingredientes: dados de entrada
    - Bolo: dados de saída
    - Receita: programa
    - Ato de fazer o bolo: processo
### Diagrama de Entidade e Relacionamento
- Ex: "Empresa"(m) emprega (n)"empregados"
- Ex: "Carro"(1) possui (n)"pneu"
- Ex: "Programa"(1) possui (n)"processos"
- OBS: No Unix, o mesmo processo pode ter mais de um programa; isto é: "Programa"(n) possui (n)"processos"

### Processamento de processos
- Com 1 núcleo, cada processo é executado unicamente por unidade de tempo
- Com mais de um núcleo, é possível executar processos simultaneamente por unidade de tempo em cada núcleo
- Atualmente, para que não seja executado um processo por vez, partes do procesos são executados em intervalos de tempo diferentes, criando a impressão de que estão sendo rodados simultaneamente, ainda que haja um único núcleo rodando. Então, com mais de um núcleo, vários processos são executados de forma significativamente rápida
- Caso um número de processos maior que o número de núcleos (mas que não seja múltiplo) sejam executados, eventualmente um núcleo acabará rodando o processo que foi já executado, em algum intervalo de tempo, por outro núcleo, exigindo a limpagem de cache,que perderá um certo desempenho, mas ainda assim roda efetivamente

### Processos do ponto de vista do usuário
- Interface linha de comando
    - O signal "prompt" seguido do nome do programa seguido de parâmetros invoca a execução do programa para tais parâmetros, colocando na tela a saída do programa
    - Normalmente, a tela é a saída padrão, portanto a execução do programa joga na própria tela da interface de linha de comando para que o usuário visualize a saída de tal programa
    - Normalmente, o teclado é a entrada padrão
    - Normalmente, a tela é a saída de erro padrão
    - Ex: Programa "Sort"

    \$ sort  
    B  
    A  
    Y  
    // CTRL+D para acabar a execução do programa e mostrar a saída na tela  
    A  
    B  
    Y  
    \$ // Após a saída, a interface volta com o símbolo prompt para permitir a   execução de novos comandos  
- Redirecionamento da entrada padrão
    - Ex: $ sort < in.txt # O símbolo "<" redireciona a entrada padrão para o arquivo
- Redirecionamento da saída padrão
    - Ex: $ sort > out.txt # O símbolo ">" redireciona a saída padrão para o arquivo
- Se um prompt estiver rodando um programa grande, a interface fica sem utilização para o símbolo prompt aparecer novamente e liberar a execução de outros comandos
- Com o símbolo "&", é possível executar em paralelo outros comandos enquanto um processo da interface com o usuário já está em execução, logo o prompt aparece rapidamente. Nessa situação, é recomendado redirecionar a saída para outros arquivos, evitando que o símbolo de prompt (e o que venha a ser digitado) se confunde com a saída do programa
- Com o símbolo |, é possível enviar a saída de um processo para a entrada de outro processo. 