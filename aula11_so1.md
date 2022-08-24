## Aula 11 - Sistemas Operacionais 1

### Resolvendo Deadlock

- Terminado: vetor de booleanos dizendo se o processo pode terminar
- Disponivel: vetor para cada tipo de recurso, contando a quantidade de recursos disponíveis de cada tipo
- Alocação: matriz processo x recurso, dizendo quantos recursos de cada tipo estão alocados a cada processo
- Pedido: matriz processo x recurso dizendo quantos recursos cada proceso está pedindo  

```
livre = disponivel

loop:
    se encontrar um processo que não esteja terminado e que seu pedido seja <= livre
        livre += alocação do processo
        marca o processo cmo terminado
    else: 
        goto fim
    goto loop

fim:
    se todos os porocessos terminados não estão em deadlock
```

- Exemplo (teste de mesa):

Terminado = (F, F, F, F)  
Disponivel = (0, 0, 1)

Alocacao

Processo|R1|R2|R3
--------|--|--|--
P1      |1 |1 |1
P2      |2 |1 |2
P3      |1 |1 |0
P4      |1 |1 |1

Pedido

Processo|R1|R2|R3
--------|--|--|--
P1      |3 |2 |1
P2      |2 |2 |1
P3      |0 |0 |1
P4      |1 |1 |1

- Passo 1: o pedido do P3 é atendido (0, 0, 1)

Terminado = (F, F, V, F)  
Livre = (1, 1, 1)

Alocacao

Processo|R1|R2|R3
--------|--|--|--
P1      |1 |1 |1
P2      |2 |1 |2
P4      |1 |1 |1


Pedido

Processo|R1|R2|R3
--------|--|--|--
P1      |3 |2 |1
P2      |2 |2 |1
P4      |1 |1 |1

- Passo 2: o processo 4 é atendido (1, 1, 1)
 
Terminado = (F, F, V, V)  
Livre = (2, 2, 2)

Alocacao

Processo|R1|R2|R3
--------|--|--|--
P1      |1 |1 |1
P2      |2 |1 |2


Pedido

Processo|R1|R2|R3
--------|--|--|--
P1      |3 |2 |1
P2      |2 |2 |1

- Passo 3: o processo 2 é atendido (2, 2, 1)

Terminado = (F, V, V, V)  
Livre = (4, 3, 4)

Alocacao

Processo|R1|R2|R3
--------|--|--|--
P1      |1 |1 |1


Pedido

Processo|R1|R2|R3
--------|--|--|--
P1      |3 |2 |1

- Passo 4: o processo 1 é atendido (3, 2, 1)

Terminado = (V, V, V, V)  
Livre = (5, 4, 5)

```
Como todos os processos terminaram, não há deadlock
```

