# programacao-paralela
Repositório criado para compartilhar a atividade avaliativa da disciplina de Sistemas Distribuídos da grade do 7º período do curso de Sistemas de Informação. **[Clique aqui para acessar o projeto no Google Colab](https://colab.research.google.com/drive/1F3GZ_ONZtwJPFC-D1lhmjrKWgBKFZ4fE?usp=sharing)**
#

# Explicando o Código

### • Configuração

É necessário incialmente utilizar o comando `!pip install mpi4py` para que a máquina do [Google Colab](https://colab.research.google.com/) receba o pacote mpi4py. Ele permite o envio e recebimento de dados em texto para diferentes processos.

### • O Código

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

### Python

```Python
import os
from time import process_time

tempo_exec=[]
tempo_teorico=[]
numeroNucleos = 1
medicaoInicial = 15
medicaoFinal = 24

print("====== " + str(numeroNucleos) + " NÚCLEOS ======")

for i in range(medicaoInicial-1, medicaoFinal):
  start_time = process_time()
  texto = (os.popen("mpirun --allow-run-as-root -np " + str(numeroNucleos) + " python chuvampi.py "+ str(i+1)))
  print(texto.read())
  stop_time = process_time()
  tempo_exec.append((i+1,(stop_time-start_time)*100))
  tempo_teorico.append((i+2, ((stop_time-start_time)*100)*2))
  print("Tempo da medição %d: %fs" % (i+1, (stop_time-start_time)*100))
print(tempo_exec)
print(tempo_teorico)
```

```
====== 1 NÚCLEOS ======

Tempo da medição 15: 0.588659s

Tempo da medição 16: 0.664543s

Tempo da medição 17: 1.045976s

Tempo da medição 18: 1.326000s

Tempo da medição 19: 1.976835s

Tempo da medição 20: 3.601016s

Tempo da medição 21: 6.780501s

Tempo da medição 22: 12.921686s

Tempo da medição 23: 26.077140s

Tempo da medição 24: 52.701332s
[(15, 0.5886589000001052), (16, 0.6645431000000812), (17, 1.0459759999999818), (18, 1.3259995000000302), (19, 1.976835099999974), (20, 3.6010160000000013), (21, 6.780500500000031), (22, 12.921686300000168), (23, 26.077140399999976), (24, 52.70133150000014)]
[(16, 1.1773178000002105), (17, 1.3290862000001624), (18, 2.0919519999999636), (19, 2.6519990000000604), (20, 3.953670199999948), (21, 7.202032000000003), (22, 13.561001000000061), (23, 25.843372600000336), (24, 52.15428079999995), (25, 105.40266300000027)]
```

### Python

```Python
import matplotlib.pyplot as plt
plt.figure(figsize=(10,7))
plt.plot([tempo_exec[i][0] for i in range(len(tempo_exec))], [tempo_exec[i][1] for i in range(len(tempo_exec))], label='Alg.Serial')
plt.plot([tempo_teorico[i][0] for i in range(len(tempo_teorico)-1)], [tempo_teorico[i][1] for i in range(len(tempo_teorico)-1)], label='Previsto')
plt.title('Tempo de execução')
plt.xlabel('Tamanho da Lista')
plt.ylabel('Segundos')
plt.legend()
plt.grid()
plt.show()
```

![image](https://user-images.githubusercontent.com/94007911/187096121-b0579b27-6121-43d4-a96d-def1fb6306df.png)

### Python

```Python
import os
from time import process_time

tempo_exec=[]
tempo_teorico=[]
numeroNucleos = 2
medicaoInicial = 15
medicaoFinal = 24

print("====== " + str(numeroNucleos) + " NÚCLEOS ======")

for i in range(medicaoInicial-1, medicaoFinal):
  start_time = process_time()
  texto = (os.popen("mpirun --allow-run-as-root -np " + str(numeroNucleos) + " python chuvampi.py "+ str(i+1)))
  print(texto.read())
  stop_time = process_time()
  tempo_exec.append((i+1,(stop_time-start_time)*100))
  tempo_teorico.append((i+2, ((stop_time-start_time)*100)*2))
  print("Tempo da medição %d: %fs" % (i+1, (stop_time-start_time)*100))
print(tempo_exec)
print(tempo_teorico)
```

```
====== 2 NÚCLEOS ======

Tempo da medição 15: 0.702228s

Tempo da medição 16: 0.939297s

Tempo da medição 17: 0.737217s

Tempo da medição 18: 1.286879s

Tempo da medição 19: 1.907728s

Tempo da medição 20: 3.104291s

Tempo da medição 21: 6.726400s

Tempo da medição 22: 12.800284s

Tempo da medição 23: 25.909203s

Tempo da medição 24: 53.226022s
[(15, 0.702227999999927), (16, 0.9392966000000058), (17, 0.737216599999968), (18, 1.286879000000063), (19, 1.9077282999999667), (20, 3.1042910000000035), (21, 6.726400100000163), (22, 12.800283999999884), (23, 25.909203000000147), (24, 53.22602219999997)]
[(16, 1.404455999999854), (17, 1.8785932000000116), (18, 1.474433199999936), (19, 2.573758000000126), (20, 3.8154565999999335), (21, 6.208582000000007), (22, 13.452800200000326), (23, 25.600567999999768), (24, 51.818406000000294), (25, 106.45204439999993)]
```

![image](https://user-images.githubusercontent.com/94007911/187096133-e9542513-0d8a-434b-aef7-322ed0edabb5.png)

