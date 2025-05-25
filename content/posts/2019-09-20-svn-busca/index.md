---
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"

date: "2019-09-20T11:43:00Z"

resources:
- name: "featured-image"
  src: "svn-busca-banner.png"
tags:
- svn

title: SVN - Busca avançada em múltiplos repositórios
---

Localizar commits pelo nome de usuário, descrição do commit e outras informações.

<!--more-->

## Sintaxe básica

```bash
svn log --search "$TERMO_DE_BUSCA" $repo
```

> **NOTA**
>
> `$repo` pode ser uma pasta local ou um endereço http remoto

## Exemplos

### Buscar em todos os repositórios

> Dependendo do tamanho do repositório pode ser um pouco lento.

```bash
TERMO_DE_BUSCA="7168181"; \
RAIZ="http://meusvn.com.br"; \
REPOS="\
$RAIZ/app-bob \
$RAIZ/app-tom \
$RAIZ/app-ana \
"; \
for repo in $REPOS; do \
  echo ">>>>> $repo <<<<<"; \
  svn log --search "$TERMO_DE_BUSCA" $repo; \
done | grep -P --color "(^>>>>>.*|$TERMO_DE_BUSCA|$)"
```

### Buscar em todos os repositórios (otimizado)

> A busca vai parar na primeira ocorrência de cada repositório. Bom para saber apenas em qual repositório houve _commit_.

```bash
TERMO_DE_BUSCA="7168181"; \
RAIZ="http://meusvn.com.br"; \
REPOS="\
$RAIZ/app-bob \
$RAIZ/app-tom \
$RAIZ/app-ana \
"; \
for repo in $REPOS; do \
  echo ">>>>> $repo <<<<<"; \
  svn log --search "$TERMO_DE_BUSCA" $repo \
  | grep -P -m 1 "$TERMO_DE_BUSCA"; \
done | grep -P --color "(^>>>>>.*|$TERMO_DE_BUSCA|$)"
```

### Otimizando ainda mais

> Bom para repositórios muito grandes ou apenas para delimitar um período no tempo.

Setar a data inicial e final na busca. Adicionar imediatamente antes do `--search` a data início e data fim, no formato abaixo:

- `-r HEAD:{2019-06-01}`; ou
- `-r {2019-08-05}:{2019-06-01}`

Exemplo:

```bash
# ex 1
svn log -r HEAD:{2019-06-01} --search "$TERMO_DE_BUSCA" $repo
# ex 2
svn log -r {2019-08-05}:{2019-06-01} --search "$TERMO_DE_BUSCA" $repo
```
