---
layout: post
title: "Fazer contas no Bash"
date: 2022-04-08 09:03:12 -0300
slug: fazer-contas-no-bash
tags: [bash, awk, bc, for, linux]
excerpt: "Algumas formas diferentes de realizar contas diretamente no terminal Bash utilizando os comandos bc, awk e funções nativas do próprio Bash."
---

Dia desses eu estava precisando coletar alguns números, saída de um comando no Linux, e somá-los. Eu poderia muito bem pegar esses números, levar para uma planilha e fazer a soma lá, mas, tendo tantas formas de realizar isso diretamente no terminal, considerei desnecessário.

Com isso, vou mostrar aqui algumas formas diferentes de realizar contas diretamente no Bash, utilizando os comandos **bc, awk e até com o próprio bash**.

Nota: Não vou mostrar grandes e complexas equações aqui, vamos focar no básico.

O primeiro `$` dos comandos é para representar a entrada de comandos no terminal bash e diferenciá-los da resposta dos comandos (que não tem `$` no início).

### Comando – bc

Uma das formas mais comuns de se realizar operações aritméticas no Linux é utilizando o comando `bc`.

E na sua forma mais simples, podemos direcionar a saída de um comando para ele:

```bash
$ echo 1+1 | bc
2
```

Uma outra forma de executar a mesma instrução (obs: não funciona em qualquer terminal):

```bash
$ bc <<< 1+1
2
```

### Função nativa do Bash (v. 3+)

O Bash nos permite realizar cálculos utilizando expansão de comandos, conforme vemos abaixo:

```bash
$ echo $((1+1))
2
```

Ok, você deve estar se perguntando: mas eu preciso listar todos os números (`1+2+3+4…+N`) sempre que eu quiser fazer uma somatória?

Não, para isso podemos realizar iterações (loops) que irão facilitar a nossa vida.

### Usando o for

Em outro momento eu posso detalhar melhor o uso da estrutura/comando `for`, mas resumindo, iremos executar um loop em uma lista para atingir o nosso objetivo.

No exemplo abaixo irei somar uma lista definida, com o resultado sendo `1+1+1+1`:

```bash
$ x=0
$ for i in 1 1 1 1
> do
>  x=$(($x+$i))
> done
$ echo $x
4
```

E aqui outra forma de escrever a mesma coisa, em uma mesma linha:

```bash
$ x=0;for i in 1 1 1 1; do x=$(($x+$i));done; echo $x
4
```

Certo, agora está melhorando!  
Vamos supor que temos um arquivo com os valores que desejamos calcular; no meu caso irá chamar `numeros.txt` e terá o conteúdo abaixo:

```text
1
2
3
4
```

Podemos usar o `for` para fazer esse cálculo da seguinte forma:

```bash
$ x=0; for i in $(cat numeros.txt); do x=$(($x+$i));done; echo $x
10
```

Dessa maneira estamos usando o comando `cat` para ler o arquivo e passar para o `for` como parâmetro.

Isso já resolve o nosso problema inicial, mas e se tivéssemos outra forma de fazer isso? Apenas para ter opções mesmo 🙂

### Comando – awk

O `awk` é um comando excepcional, que contém uma imensa gama de opções para manipular dados, mas no momento vamos nos ater a realizar a mesma coisa que fizemos com o último `for`:

```bash
$ awk '{x+=$1;} END{print x;}' numeros.txt
10
```

Explicando o comando acima: estamos realizando um loop pelo arquivo `numeros.txt` e adicionando à variável `x` o valor atual mais o valor da linha corrente, e ao final exibindo o valor.

Espero que tenha gostado dessas dicas!  
Até a próxima 😀
