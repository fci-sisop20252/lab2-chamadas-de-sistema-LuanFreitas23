# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: __7___ syscalls
- ex1b_write: __8___ syscalls

**2. Por que há diferença entre printf() e write()?**

```
[Há diferença, pois o printf é capaz de escrever textos para o usuário, formatar dados e é utilizado para casos que não seja necessário uma crítica de desempenho. E o write é capaz de escrever, além de texto, dados binários, controlar quando enviar os dados e possuir um comportamento previsível.]
```

**3. Qual implementação você acha que é mais eficiente? Por quê?**

```
[O write, porque cada chamada gera uma syscall direta, sem depender de buffer, então o resultado sai sempre no mesmo momento.]
```

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: __3___
- Bytes lidos: __127___

### Comando strace:
```bash
strace -e open,read,close ./ex2_leitura
```

### Análise

**1. Por que o file descriptor não foi 0, 1 ou 2?**

```
[Porque os descritores 0, 1 e 2 já estão reservados para entrada padrão(stdin), saída padrão (stdout) e erro padrão (stderr). O próximo arquivo aberto pelo processo recebe o próximo número livre, que normalmente é o 3.]
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
[Porque a chamada read() retornou exatamente a quantidade de bytes do arquivo e, se chamasse read() de novo, o retorno seria 0, indicando fim de arquivo]
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: __24___ (esperado: 25)
- Caracteres: __1299___
- Chamadas read(): __21___
- Tempo: __0.000100___ segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |   82            | 0.000224  |
| 64          |   21            | 0.000100  |
| 256         |   6             | 0.000088  |
| 1024        |   2             | 0.000058  |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
[Quanto menor o buffer, mais chamadas read() são necessárias para ler o arquivo todo.]
```

**2. Como você detecta o fim do arquivo?**

```
[Quando read() retorna 0, indicando fim do arquivo.]
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: __1364___
- Operações: __6___
- Tempo: __0.000180___ segundos
- Throughput: __7400.17___ KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ ] Idênticos [x] Diferentes []

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
[Porque o write() pode escrever menos bytes do que foi pedido, dependendo do estado do sistema. Se não conferirmos, podemos perder dados na cópia ou copiar só parcialmente o conteúdo lido.]
```

**2. Que flags são essenciais no open() do destino?**

```
[O_WRONLY - para abrir em modo escrita   O_CREAT - para criar o arquivo que não exista   O_TRUNC - para truncar o arquivo se ele já existir, garantindo que ele não fique no lixo antigo.]
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
[As syscalls são pontos em que o programa sai do modo usuário e pede serviço ao kernel, como abrir, ler, escrever ou fechar arquivos. Elas marcam claramente a fronteira entre código do usuário e operações controladas pelo sistema operacional.]
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
[Os file descriptors funcionam como “identificadores” que o kernel usa para acessar arquivos, sockets ou dispositivos. Eles permitem que o programa manipule diferentes recursos de I/O de forma unificada e controlada.]
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
[Buffers pequenos geram muitas syscalls, aumentando o overhead. Buffers grandes reduzem o número de chamadas e melhoram a performance, mas usam mais memória. O ideal é escolher um tamanho que balanceie uso de memória e velocidade.]
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** __cp do sistema___

**Por que você acha que foi mais rápido?**

```
[Porque o cp é altamente otimizado: usa buffers maiores, técnicas internas do kernel e menos chamadas de sistema, reduzindo o overhead em comparação com nosso programa simples.]
```

---

## Entrega

Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```