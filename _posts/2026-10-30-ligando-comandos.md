---
layout: post
title: "Ligando comandos: Redirecionadores e Pipes no Bash"
date: 2026-10-30 21:02:43 -0300
slug: ligando-comandos
tags: [linux, bash, shell, conectores, redirecionadores, terminal]
excerpt: "Domine o uso prático de redirecionadores e conectores no Bash: aprenda a trabalhar com >, >>, |, <, <<< e << para automatizar tarefas no terminal."
---

Um dos pilares da filosofia Unix é simples e poderoso: *faça programas que realizem apenas uma tarefa, façam bem-feita e que consigam trabalhar em conjunto através de texto plano*.

No dia a dia do terminal, é muito comum precisarmos capturar o resultado de um comando para salvar em arquivo, alimentar outro processo em memória ou injetar variáveis sem criar scripts complexos.

Para isso, o Bash oferece **redirecionadores de fluxo** (entrada, saída e erro) e **conectores (pipes)**. Vamos explorar os principais com exemplos práticos.

---

### 1. Redirecionador de Saída Padrão — Sobrescrever (`>`)

O operador `>` direciona a saída padrão (`stdout`) de um comando diretamente para um arquivo.
* Se o arquivo **não existir**, ele é criado.
* Se o arquivo **já existir**, seu conteúdo anterior é completamente apagado (truncado) e substituído pelo novo.

```bash
$ uptime > status.txt
$ cat status.txt
 14:30:15 up 14 days,  4:12,  2 users,  load average: 0.12, 0.08, 0.05
```

Se executarmos outro comando apontando para o mesmo arquivo com `>`, o conteúdo antigo dá lugar ao novo:

```bash
$ hostname > status.txt
$ cat status.txt
srv-producao-01
```

---

### 2. Redirecionador de Saída Padrão — Anexar (`>>`)

Quando você precisa registrar históricos, criar arquivos de *log* ou manter o que já existia no arquivo, utiliza-se o operador `>>`.
* Se o arquivo **não existir**, ele é criado.
* Se o arquivo **já existir**, o novo conteúdo é anexado ao final (*append*), preservando as linhas anteriores.

```bash
$ echo "Iniciando rotina de backup..." > deploy.log
$ echo "Executado em: $(date '+%Y-%m-%d %H:%M:%S')" >> deploy.log
$ echo "Status: OK" >> deploy.log
$ cat deploy.log
Iniciando rotina de backup...
Executado em: 2026-10-30 21:05:00
Status: OK
```

---

### 3. Conector de Comandos (`|` — Pipe)

O caractere *pipe* (`|`) conecta a saída padrão (`stdout`) do comando à esquerda diretamente à entrada padrão (`stdin`) do comando à direita.

Tudo acontece **em memória**, sem a necessidade de gravar arquivos temporários em disco. É a ferramenta essencial para encadear filtros.

```bash
# Descobrir quantos processos do nginx estão em execução:
$ ps aux | grep [n]ginx | wc -l
4

# Listar os 5 maiores diretórios dentro de /var/log:
$ du -sh /var/log/* 2>/dev/null | sort -hr | head -n 5
2.4G	/var/log/journal
412M	/var/log/syslog
120M	/var/log/nginx
45M	/var/log/auth.log
12M	/var/log/dpkg.log
```

---

### 4. Redirecionamento de Entrada Padrão (`<`)

Enquanto o `>` envia dados para um arquivo, o `<` faz o caminho inverso: alimenta a entrada padrão (`stdin`) de um comando com o conteúdo de um arquivo.

É indispensável quando o comando não aceita receber nomes de arquivos como argumentos diretamente, ou quando você deseja isolar o processamento do texto.

```bash
# O comando wc lendo diretamente o arquivo exibe o nome do arquivo na saída:
$ wc -l ips.txt
42 ips.txt

# Mas usando o redirecionador de entrada <, ele recebe apenas o fluxo de dados:
$ wc -l < ips.txt
42
```

> **Dica de Script:** A segunda forma é ideal quando você quer armazenar apenas o número em uma variável no Bash, sem precisar usar `cut` ou `awk` para remover o nome do arquivo:
> ```bash
> TOTAL_IPS=$(wc -l < ips.txt)
> echo "Total de IPs encontrados: $TOTAL_IPS"
> ```

---

### 5. Here-String (`<<<`)

O operador `<<<` injeta uma string ou o conteúdo de uma variável diretamente na entrada padrão (`stdin`) do comando.

Ele elimina a necessidade de fazer `echo "$variavel" | comando`, evitando a criação de um processo extra (*subshell*) na tabela do sistema operacional.

```bash
# Forma tradicional com pipe (cria subprocesso desnecessário):
$ echo "192.168.1.100" | cut -d'.' -f4
100

# Usando Here-String (muito mais rápido e limpo):
$ cut -d'.' -f4 <<< "192.168.1.100"
100
```

Outro exemplo comum para transformar caracteres com o utilitário `tr`:

```bash
$ PACOTE="kernel-core-6.8.0.rpm"
$ tr '[:lower:]' '[:upper:]' <<< "$PACOTE"
KERNEL-CORE-6.8.0.RPM
```

---

### 6. Here-Document (`<<`)

O *Here-Document* (ou `heredoc`) envia um bloco inteiro de texto com múltiplas linhas para a entrada padrão de um comando, delimitado por um marcador de início e fim (por convenção, utiliza-se `EOF`, `FIM` ou `TEXTO`).

É excelente para gerar arquivos de configuração ou scripts sem precisar de dezenas de comandos `echo`:

```bash
$ cat << 'EOF' > /etc/resolv.conf
# Configuração de DNS gerada automaticamente
nameserver 1.1.1.1
nameserver 8.8.8.8
options timeout:2 attempts:3
EOF
```

> **Dica sobre aspas no delimitador:**
> * Se você usar `<< EOF` (sem aspas), o Bash **irá interpretar** variáveis como `$USER`, `$(hostname)` e comandos no meio do texto.
> * Se você usar `<< 'EOF'` (com aspas simples), o Bash **não interpretará** nada, gravando o texto de forma literal.

---

### Bônus: Redirecionando Saída de Erro (`2>` e `&>`)

No terminal, existem três canais fundamentais:
* `0`: Entrada padrão (`stdin`)
* `1`: Saída padrão (`stdout`)
* `2`: Saída de erro padrão (`stderr`)

Quando um comando gera um erro, o operador `>` normal não captura a mensagem (pois ele atua apenas no canal `1`). Para controlar mensagens de erro:

```bash
# 1. Descartar mensagens de erro (enviando para o buraco negro /dev/null):
$ find /etc -name "*.conf" 2> /dev/null

# 2. Salvar saídas normais em um arquivo e erros em outro:
$ ./meu_script.sh > execucao.log 2> erros.log

# 3. Redirecionar tudo (stdout e stderr) para o mesmo arquivo:
$ ./meu_script.sh &> geral.log
```

---

### Conclusão

Entender a fundo o funcionamento dos redirecionadores e conectores no terminal auxilia enormemente na produtividade diária, reduz a quantidade de linhas de código em scripts e, o mais importante: permite combinar utilitários simples e confiáveis para resolver problemas complexos com elegância.
