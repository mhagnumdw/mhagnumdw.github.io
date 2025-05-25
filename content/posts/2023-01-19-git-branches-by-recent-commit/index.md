---
author-id: mhagnumdw
categories:
- git
date: "2023-01-19T16:18:34Z"
feature-img: assets/img/posts/git-branches-by-recent-commit/banner.png
image: assets/img/posts/git-branches-by-recent-commit/banner.png
tags:
- git
thumbnail: assets/img/posts/git-branches-by-recent-commit/banner.png
title: 'Git: listar as branches por commit mais recente'
---

Listar as branches por commit mais recente. Útil para ter um overview sobre quais branches estão mais desatualizadas.

<!--more-->

> A ideia foi retirada da referência mais abaixo, mas a função está um pouco modificada visando melhorias.

## Configurar o alias

Vamos faze por meio de um alias.

```bash
git config --edit --global
# ou se quiser editar no vscode
GIT_EDITOR='code --wait' git config --global --edit
```

Adicionar dentro da seção `[alias]` o código da função `recentb`, conforme abaixo

```config
[alias]
    # https://stackoverflow.com/questions/5188320/how-can-i-get-a-list-of-git-branches-ordered-by-most-recent-commit#5972362
    # ATTENTION: All aliases prefixed with ! run in /bin/sh make sure you use sh syntax, not bash/zsh or whatever
    recentb = "!r() { \
        refbranch=$1; \
        count=$2; \
        refs=$3; \
        defaultBranch=$(git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'); \
        git for-each-ref \
        --sort=-committerdate \
        ${refs:-refs/remotes} \
        --format='%(refname:short)|%(HEAD)%(color:yellow)%(refname:short)|%(color:bold green)%(committerdate:relative)|%(color:blue)%(subject)|%(color:magenta)%(authorname)%(color:reset)' \
        --color=always \
        --count=${count:-200} \
        | while read line; do \
            branch=$(echo \"$line\" | awk 'BEGIN { FS = \"|\" }; { print $1 }' | tr -d '*'); \
            ahead=$(git rev-list --count \"${refbranch:-origin/${defaultBranch}}..${branch}\"); \
            behind=$(git rev-list --count \"${branch}..${refbranch:-origin/${defaultBranch}}\"); \
            colorline=$(echo \"$line\" | sed 's/^[^|]*|//'); \
            echo \"$ahead|$behind|$colorline\" | awk -F'|' -vOFS='|' '{$5=substr($5,1,70)}1' ; \
        done \
        | ( echo \"ahead|behind|branch|lastcommit|message|author\" && cat) \
        | column -ts'|'; \
    }; r"
```

Observar que a função/alias `recentb` recebe alguns parâmetros.

## Exemplos de uso

### Sem uma branch de referência

Vai assumir que a branch de referência é a branch default, geralmente `master` ou `main`.

```bash
git recentb
```

![git-recentb]({{ site.baseurl }}/assets/img/posts/git-branches-by-recent-commit/git-recentb.png)

### Especificando uma branch de referência

```bash
git recentb origin/post-memoria-jvm-container-docker
```

![git-recentb]({{ site.baseurl }}/assets/img/posts/git-branches-by-recent-commit/git-recentb-ref-branch.png)

### Especificando a quantidade de branches e o refs

```bash
git recentb origin/master 10 refs/tags
```

Nesse exemplo, `origin/master` é a branch de referência, vai trazer no máximo 10 referências e vai inspecionar `refs/tags` (por padrão inspeciona `refs/remotes`).

## Referências

- <https://stackoverflow.com/questions/5188320/how-can-i-get-a-list-of-git-branches-ordered-by-most-recent-commit#5972362>
