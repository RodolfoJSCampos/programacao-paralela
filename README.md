# programacao-paralela
Repositório criado para compartilhar a atividade avaliativa da disciplina de Sistemas Distribuídos da grade do 7º período do curso de Sistemas de Informação.

**[Clique aqui para acessar o projeto no Google Colab](https://colab.research.google.com/drive/1F3GZ_ONZtwJPFC-D1lhmjrKWgBKFZ4fE?usp=sharing)**
#

# Explicando o Código

### • Configuração

É necessário incialmente utilizar o comando `!pip install mpi4py` para que a máquina do [Google Colab](https://colab.research.google.com/) receba o pacote mpi4py. Ele permite o envio e recebimento de dados em texto para diferentes processos.

### • O Código

#### Algoritmo Serial:
Primeiramente é calculado o tempo de execução do algoritmo serial, para isso foi utilizado o [modelo fornecido](https://colab.research.google.com/drive/1dzcm7nBqqzQAtD6XOZWmCvWcB6HCEd0Z?usp=sharing#scrollTo=1qlmCkWb_VqV) na explanação no problema, conforme mostrado abaixo:

```Python
%%writefile alg_serial.py
#Importação das bibliotecas
import os
import random
import sys

random.seed(1) #Utilizado aqui para efeito de estudo, pois os mesmos valores
               #serão sorteados sempre, facilitando a explicação.

#Recebendo a lista de valores por parâmetro do sistema
tam_lista = int(sys.argv[1])
SOMA = 10 #Valor de soma atribuído - Poderia ser qualquer valor.

combinacoes = [] #Esta lista guarda todas as combinações de somas possíveis.

#Gerando as combinações de números da lista possíveis, dependendo do tamanho
#passado por parâmetro (tam_lista).
#As combinações de números da lista será de ordem "2^tam_lista", iniciando
#em: 0000...0 até 1111...1. 
print('\nCombinações: ')
for i in range(0, 2**tam_lista):
  x = bin(i)
  #print(x)
  combinacoes.append(x.split('b')[1].zfill(tam_lista))
print(combinacoes)
print()

#Serão gerados aleatóriamente uma quantidade de números definida por 'tam_lista'
sorteio = ''
for x in range(0,tam_lista):
    sorteio += str(random.randint(1,10)) + ' '
sorteio = sorteio[:len(sorteio)-1].split(' ')
print('\nNº sorteados:')
print(sorteio)

print('\nCombinações que apresentam a soma %d' %SOMA)
total = 0
for l in combinacoes:
  soma = 0
  fatores = list(l)
  for pos in range(0, len(l)):
    soma += int(fatores[pos]) * int(sorteio[pos])
  if soma == SOMA:
    total += 1
    print(sorteio)
    print(fatores)
    print()
print('Total de combinações com a soma %d: %d' % (SOMA, total))
```

#### Gravando o tempo do Algoritmo Serial:
O código abaixo coleta o tempo para um número predefinido de medições.

```Python
import time as t
import os
tempo_exec_serial=[]
tempo_teorico_serial=[]

print("============ SERIAL ===========")

for i in range(15, 24):
  tempo_atual = t.time()
  texto = (os.popen("python alg_serial.py "+str(i)+' >> /dev/null'))
  print(texto.read())
  tempo_final = t.time()
  tempo_final -= tempo_atual
  tempo_exec_serial.append((i,tempo_final))
  print("Tempo da medição %d: %fs" % (i, (tempo_final)))
print("")
print("================================")
```

#### Saída do Algoritmo Serial:
A saída para uma amostragem começando em 15 medições e indo até 23 medições pode ser vista abaixo:

```
============ SERIAL ===========

Tempo da medição 15: 0.404765s

Tempo da medição 16: 0.716470s

Tempo da medição 17: 1.526503s

Tempo da medição 18: 2.839469s

Tempo da medição 19: 5.733772s

Tempo da medição 20: 12.231317s

Tempo da medição 21: 24.658842s

Tempo da medição 22: 54.274817s

Tempo da medição 23: 110.447614s

================================
```

#### Algoritmo Paralelizado:
Utilizando o pacote `mpi4py` foi possível desenvolver o código abaixo, que recebe como parâmetro um número referente ao número de processos desejados para a paralelização `size = comm.Get_size()`, e assim como no algoritmo serial, recebe também um número referente tamanho das medições que serão calculadas `tam_medicoes = int(sys.argv[1])`. Os processos são então inicializados e têm seus PID's definidos aqui por `rank = comm.Get_rank()`. Em seguida são utilizados laços de repetições e verificações para separar o tamanho da amostragem baseado no tamanho das medições predefinido `size`, para que então o processo raíz (PID = 0) encaminhe qual o espaço amostral de cada processo baseado nos seus PID's através da chamada `comm.send(medicao, i)`. Os demais processos recebem seus espaços amostrais de medição através da chamada `medicao = comm.recv()`.

De forma similar ao Algoritmo Serial são feitos os cálculos e as comparações necessárias e então o processo raíz aguarda o recebimento dos resultados dos demais processos através de um laço de repetição da chamada `solucao.append(comm.recv())`, que armazena os resultados em um vetor de `solucao`. Os demais processos enviam seus resultados para o processo raíz (PID = 0) através da chamada `comm.send(solucao, 0)`. Após todos os processos eviarem suas soluções, o processo zero encerra a execução com o vetor `solucao` preenchido com todas as soluções encontradas pelos processos.

```Python
%%writefile chuvampi.py
import os
import random
import sys
from mpi4py import MPI

comm = MPI.COMM_WORLD
size = comm.Get_size()
rank = comm.Get_rank()
combinacoes = []
solucao = []
total = 0
soma = 0
medicao = ''

tam_medicoes = int(sys.argv[1])
soma_usuario = 8

if(rank == 0):
  for x in range(0,tam_medicoes):
    medicao += str(random.randint(1,10)) + ' '
  medicao = medicao[:len(medicao)-1].split(' ')
  for i in range(1,size):
    comm.send(medicao, i)
else:
  medicao = comm.recv()

for i in range(int(rank*(((2**tam_medicoes)/size))), int(((rank+1)*(2**tam_medicoes))/size)):
  x = bin(i)
  combinacoes.append(x.split('b')[1].zfill(tam_medicoes))

for l in combinacoes:
  fatores = list(l)
  for pos in range(0, len(l)):
    soma += int(fatores[pos]) * int(medicao[pos])
  if soma == soma_usuario:
    solucao.append(fatores)
  soma = 0

if(rank == 0):
  for i in range(size-1):
    solucao.append(comm.recv())
else:
  comm.send(solucao, 0)
```

#### Gravando o tempo do Algoritmo Paralelizado:
De forma similar ao Algoritmo Serial, o código abaixo coleta o tempo para um número predefinido de medições.

```Python
import os
import time as t

tempo_exec_paralelo=[]
tempo_teorico_paralelo=[]
numeroNucleos = 2
medicaoInicial = 15
medicaoFinal = 23

print("========== " + str(numeroNucleos) + " NÚCLEOS ==========")

for i in range(medicaoInicial-1, medicaoFinal):
  tempo_atual = t.time()
  texto = (os.popen("mpirun --allow-run-as-root -np " + str(numeroNucleos) + " python alg_paralelo.py "+ str(i+1)))
  print(texto.read())
  tempo_final = t.time()
  tempo_final -= tempo_atual
  tempo_exec_paralelo.append((i+1,tempo_final))
  print("Tempo da medição %d: %fs" % (i+1, (tempo_final)))
print("")
print("===============================")
```

#### Saída do Algoritmo Paralelizado:
Devido à uma limitação da infraestrutura de softwares do Google Colab, só é possível paralelizar o processo em dois núcleos. A saída para uma amostragem começando em 15 medições e indo até 23 medições para uma paralelização de 2 núcleos pode ser vista abaixo:

```
========== 2 NÚCLEOS ==========

Tempo da medição 15: 0.654010s

Tempo da medição 16: 1.012584s

Tempo da medição 17: 1.739444s

Tempo da medição 18: 3.125001s

Tempo da medição 19: 6.217459s

Tempo da medição 20: 12.520230s

Tempo da medição 21: 26.260201s

Tempo da medição 22: 54.468410s

Tempo da medição 23: 113.167438s

===============================
```

### Plotando o Gráfico
De forma similar ao [modelo fornecido](https://colab.research.google.com/drive/1dzcm7nBqqzQAtD6XOZWmCvWcB6HCEd0Z?usp=sharing#scrollTo=1qlmCkWb_VqV) para a explanação do problema, foi utilizado o código abaixo para plotagem do gráfico comparatório, inserindo então a curva de tempo por número de medições do Algoritmo Paralelizado.

```Python
import matplotlib.pyplot as plt
plt.figure(figsize=(10,7))
plt.plot([tempo_exec_serial[i][0] for i in range(len(tempo_exec_serial))], [tempo_exec_serial[i][1] for i in range(len(tempo_exec_serial))], label='Alg.Serial')
plt.plot([tempo_exec_paralelo[i][0] for i in range(len(tempo_exec_paralelo))], [tempo_exec_paralelo[i][1] for i in range(len(tempo_exec_paralelo))], label='Alg.Paralelo')
plt.title('Tempo de execução')
plt.xlabel('Número de Medições')
plt.ylabel('Segundos')
plt.legend()
plt.grid()
plt.show()
```

![image](https://user-images.githubusercontent.com/94007911/187323310-99b6acad-a5ac-4fc1-8819-f369bf08721c.png)

### Analisando os Resultados
É evidente que os dois modelos de algoritmos obtiveram tempos de processamento iguais ou muito próximos. Isso se deu devido à uma limitação estabelecida pelo Google Colab, fazendo com que a curva de tempo do modelo paralelizado se comportasse de maneira semelhante ao modelo serial. A implementação desse código em uma máquina sem restrições de uso de núcleos de processamento e virtualização certamente resultaria em uma diferença clara no tempo de processamento para o modelo paralelizado, visto que múltiplos núcleos estariam realizando simultaneamente os cálculos que apenas um núcleo estaria realizando de forma serial no modelo do Algoritmo Serial.
