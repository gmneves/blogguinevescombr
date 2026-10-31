---
layout: post
title: "Conheça o super xargs!"
date: 2026-10-29 21:19:53 -0300
slug: conheca-o-super-xargs
tags: [linux, bash, xargs, pipe, comando, script]
excerpt: "Funcionamento básico do comando xargs para transformar a saída padrão de um comando em argumentos para outro."
---

O `xargs` é uma ferramenta que permite utilizar a saída padrão de um comando como argumentos de entrada para outro comando.

Em outro momento farei um artigo falando sobre como isso funciona; por hora, só precisamos saber que o pipe (`|`) faz a mágica de pegar o resultado do comando da esquerda e passar como argumento para o comando da direita.

Vamos a exemplos práticos!

> Nota: Claro que existem outras formas de resolver esse problema no terminal, mas o foco agora é o `xargs` :-)

### Criando múltiplos diretórios de uma vez

Vamos criar 3 diretórios de uma vez:

```bash
$ echo "primeiro segundo terceiro" | xargs mkdir
$ ls -l
drwxr-xr-x 2 usuario usuario 4096 out 29 21:15 primeiro
drwxr-xr-x 2 usuario usuario 4096 out 29 21:15 segundo
drwxr-xr-x 2 usuario usuario 4096 out 29 21:15 terceiro
```

Observe que nessa execução o `xargs` passou a lista toda como argumentos para o `mkdir`.

### Limitando a quantidade de argumentos por execução (`--max-args`)

Agora vamos pegar uma lista e dividir em pares, informando ao `xargs` que queremos passar `2` argumentos por execução do comando `echo`:

```bash
$ echo "1 2 3 4 5 6" | xargs --max-args 2 echo "Par:"
Par: 1 2
Par: 3 4
Par: 5 6
```

### Posicionando o argumento com `-I{}`

Supondo que queremos compactar os arquivos de um diretório com o `zip`, de forma isolada.

Vamos começar usando o `ls -1` para listar apenas os nomes dos arquivos em uma única coluna:

```bash
$ ls -1 /home/usuario/teste
1.txt
2.txt
3.txt
4.txt
```

Agora vamos passar o resultado desse comando para o `xargs`, informando que queremos uma execução por arquivo e usando os sinais de chaves (`{}`) como referência para a entrada dos arquivos, já que teremos que usar a mesma referência duas vezes e não somente ao final do comando. O comando que queremos executar será o `zip arquivo.zip arquivo`:

```bash
$ ls -1 /home/usuario/teste | xargs --max-args 1 -I{} zip {}.zip {}
```

Ao reexecutar o `ls`:

```bash
$ ls -1 /home/usuario/teste
1.txt
1.txt.zip
2.txt
2.txt.zip
3.txt
3.txt.zip
4.txt
4.txt.zip
```

### Execução em paralelo (`-P`)

Outra opção muito útil para o comando é o argumento `-P <número de processos>`.  
Nesse caso, você define qual o máximo de processos paralelos que o `xargs` irá disparar, sendo `0` sem limites.

No nosso exemplo, o uso do `-P 0` criaria 4 processos de `zip` paralelos. Por padrão, ele cria apenas `1` por vez.

> **IMPORTANTE:** Muito cuidado ao usar o `-P 0`; ele irá instruir o `xargs` a criar o maior número possível de execuções paralelamente, o que pode esgotar recursos do ambiente.

### Conclusão

O `xargs` é uma ferramenta extremamente útil quando utilizada em combinação com outros comandos para agilizar execuções *in-line*, sem a necessidade de criação de scripts grandes, mantendo a simplicidade e o controle fino das execuções, além de ser essencial para comandos que não recebem nativamente argumentos via pipe.
