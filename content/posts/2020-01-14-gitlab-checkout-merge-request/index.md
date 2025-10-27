---
title: 'GitLab: checkout merge request'
date: "2020-01-14T19:33:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
avatarURL: "/images/authors/dwouglas.jpg"
resources:
- name: "featured-image"
  src: "banner.jpg"
categories: ["GitLab"]
tags: ["checkout", "merge request"]
---

Realizar o checkout de um merge request do GitLab.

<!--more-->

## Opção 1: criar uma referência para o merge request

```bash
git fetch origin merge-requests/MERGE_REQUEST_ID/head:REF_BRANCH_NAME
# MERGE_REQUEST_ID = ID do Merge Request
# REF_BRANCH_NAME = um nome qualquer
```

Exemplo

```bash
git fetch origin merge-requests/118/head:MR118
git checkout MR118
```

> **Checando a referência:** `git show-ref | grep MR118`
>
> **Deletando a referência:** `git branch -D MR118`

## Opção 2: adicionando uma nova referência ao remote

É possível editando o arquivo `.git/config` ou por meio do comando `git config -e`. Vamos fazer por meio do comando.

```bash
# abrir as configurações do git no editor padrão
git config -e
```

Dentro da seção `[remote "origin"]` adicionar na última linha o trecho abaixo

```text
fetch = +refs/merge-requests/*/head:refs/remotes/origin/merge-requests/*
```

ficando algo parecido com

```text
[remote "origin"]
url = git@git.doinio.com:mhagnumdw/dummy.git
fetch = +refs/heads/*:refs/remotes/origin/*
fetch = +refs/merge-requests/*/head:refs/remotes/origin/merge-requests/*
```

então, salvar o arquivo.

Em seguida fazer o checkout do merge request

```bash
# atualizar as referências (vão aparecer as referências de merge request)
git fetch origin
# checkout do merge request (ID do merge request ao final)
git checkout origin/merge-requests/118
```

## Referências

- <https://docs.gitlab.com/ee/user/project/merge_requests/reviewing_and_managing_merge_requests.html#checkout-locally-by-modifying-gitconfig-for-a-given-repository>
- <https://stackoverflow.com/questions/44992512/git-how-to-checkout-merge-request-locally-and-create-new-local-branch#44992513>
