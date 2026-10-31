---
layout: post
title: "Como testar conexões sem telnet"
date: 2022-04-18 15:27:09 -0300
slug: como-testar-conexoes-sem-telnet
tags: [bash, rede, tcp, firewall, linux, dicas]
excerpt: "Uma alternativa nativa do Bash (/dev/tcp) para testar conexões de rede e portas entre dois pontos sem precisar de telnet, netcat e afins."
---

Vamos a mais uma dica, dessa vez vou mostrar uma alternativa para testar conexão entre 2 pontos sem utilizar executáveis como `telnet`, `netcat` e afins.

Para isso vamos fazer uso de um *"device file"* nativo chamado `/dev/tcp`.

Utilizando esse método, podemos observar alguns comportamentos, os quais irei detalhar abaixo.

Vamos usar como exemplo um teste simples de conexão com este site, na porta `443`:

```bash
$ >/dev/tcp/guineves.com.br/443
```

### Conexão OK

Caso, imediatamente após dar um Enter, o cursor retorne para o terminal sem mensagens, a conexão foi estabelecida com sucesso.

Isso pode ser confirmado verificando o *"exit code"* da última execução e vendo o retorno `0`, que significa execução sem erros:

```bash
$ echo $?
0
```

### Conexão recusada

A seguinte informação nos mostra, de forma geral, que a conexão chegou até o host, mas que a porta não está aberta:

```text
bash: connect: Connection refused
bash: /dev/tcp/guineves.com.br/443: Connection refused
```

Nesse caso o retorno do comando é `1`, indicando um erro:

```bash
$ echo $?
1
```

### Timeout

Agora, caso o comando prenda o terminal, é sinal de que a tentativa de conexão não chega no destino, sugerindo, entre outros motivos, um bloqueio de firewall.

Ao final do timeout (aqui ele leva 2 minutos para acontecer), a mensagem abaixo é exibida e o código de retorno também é `1`:

```text
bash: connect: Connection timed out
bash: /dev/tcp/guineves.com.br/443: Connection timed out
```

E aqui vai uma dica dentro da dica 😀

Podemos fazer uso do comando `timeout` para reduzir essa espera e um condicional *in-line* para analisar o retorno e indicar se houve sucesso ou falha:

```bash
$ timeout 5 bash -c ">/dev/tcp/guineves.com.br/443" && echo "Conexão OK" || echo "Conexão Falhou"
```

Essa técnica pode ser utilizada para outros casos, e nada impede que você faça a análise do `"$?"` e a tratativa isolada.

Até a próxima!
