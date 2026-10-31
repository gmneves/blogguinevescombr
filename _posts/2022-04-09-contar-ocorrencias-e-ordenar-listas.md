---
layout: post
title: "Contar ocorrências e ordenar listas"
date: 2022-04-09 11:48:21 -0300
slug: contar-ocorrencias-e-ordenar-listas
tags: [awk, cut, sort, uniq, logs, linux]
excerpt: "Como usar um conjunto de comandos (awk, cut, sort e uniq) no terminal para agregar quantidades em logs e mostrar uma lista ordenada."
---

Mais uma vez, um tipo de coleta que me vejo frequentemente fazendo e venho compartilhar com você.

Nessa dica, irei mostrar como podemos usar um conjunto de comandos para agregar quantidades e nos mostrar uma lista ordenada.

Vamos supor que temos o arquivo de log abaixo, já filtrado com o conteúdo de interesse:

```text
2022-04-09 10:00:01 String interessante
2022-04-09 10:00:15 String interessante
2022-04-09 10:00:30 String interessante
2022-04-09 10:01:01 String interessante
2022-04-09 10:01:12 String interessante
2022-04-09 10:01:30 String interessante
2022-04-09 10:01:45 String interessante
2022-04-09 10:01:50 String interessante
2022-04-09 10:02:07 String interessante
2022-04-09 10:02:18 String interessante
2022-04-09 10:02:23 String interessante
2022-04-09 10:02:42 String interessante
2022-04-09 10:02:48 String interessante
2022-04-09 10:02:55 String interessante
2022-04-09 10:03:08 String interessante
2022-04-09 10:03:16 String interessante
2022-04-09 10:03:32 String interessante
2022-04-09 10:03:44 String interessante
2022-04-09 10:04:25 String interessante
```

Uma informação importante que eu quero nesse caso é saber quantas vezes por minuto a `"String interessante"` apareceu.

Para isso, primeiro vamos verificar como obter apenas o horário do log, utilizando o comando `awk`:

```bash
$ awk '{print $2}' log.out
```

O separador padrão do `awk` é o espaço, por isso não foi necessário informar no comando.

A saída desse comando é a seguinte:

```text
10:00:01
10:00:15
10:00:30
10:01:01
10:01:12
10:01:30
10:01:45
10:01:50
10:02:07
10:02:18
10:02:23
10:02:42
10:02:48
10:02:55
10:03:08
10:03:16
10:03:32
10:03:44
10:04:25
```

Em seguida, queremos isolar somente os campos de hora e minuto. Poderia fazer isso com o `awk` também, mas para variar um pouco, irei utilizar o comando `cut`:

```bash
$ awk '{print $2}' log.out | cut -d ':' -f 1,2
```

No comando acima, estamos passando a saída do primeiro comando (`awk`) para o segundo comando (`cut`) com o auxílio do direcionador pipe (`|`).  
Para o `cut`, estou informando o delimitador `':'` e informando que quero os campos `1` e `2`.

Saída do comando, até o momento:

```text
10:00
10:00
10:00
10:01
10:01
10:01
10:01
10:01
10:02
10:02
10:02
10:02
10:02
10:02
10:03
10:03
10:03
10:03
10:04
```

O que queremos agora é agregar as linhas e contar quantas ocorrências temos para cada conjunto de `hora:minuto`.

Vale uma observação aqui:  
Para usar o comando `uniq` precisamos que a lista esteja ordenada e, nesse caso, por conta da natureza do log, isso já acontece.  
Para todos os efeitos, irei ordenar a lista de qualquer forma, com o comando `sort`:

```bash
$ awk '{print $2}' log.out | cut -d ':' -f 1,2 | sort | uniq -c
```

```text
3 10:00
5 10:01
6 10:02
4 10:03
1 10:04
```

E por fim, podemos ordenar essa lista do menor para o maior, utilizando novamente o comando `sort`:

```bash
$ awk '{print $2}' log.out | cut -d ':' -f 1,2 | sort | uniq -c | sort -n
```

```text
      1 10:04
      3 10:00
      4 10:03
      5 10:01
      6 10:02
```

Caso queira inverter a lista, basta utilizar o argumento `-r` no último `sort`:

```bash
$ awk '{print $2}' log.out | cut -d ':' -f 1,2 | sort | uniq -c | sort -nr
```

```text
      6 10:02
      5 10:01
      4 10:03
      3 10:00
      1 10:04
```

Espero que essa dica tenha sido esclarecedora e principalmente que lhe seja útil!

Até a próxima!
