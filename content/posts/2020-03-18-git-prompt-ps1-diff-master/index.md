---
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"

date: "2020-03-18T09:34:00Z"

resources:
- name: "featured-image"
  src: "git-prompt-ps1-diff-master.png"
tags:
- git
- diff

title: Git - Quantidade de commits atrás/frente em relação ao master no prompt
---

Exibir no prompt, por meio do `PS1`, a quantidade de commits atrás e na frente em relação a branch `origin/master`.

<!--more-->

No prompt, dentro de um repositório Git, teremos algo parecido com: `[user@host folder] (Atras do origin/master por 1 commit e na frente por 2)$`

## Editar `~/.bashrc`

### Adicionar a função abaixo

```bash
_git_status_relation_to_origin_master() {
  if git rev-parse --is-inside-work-tree > /dev/null 2>&1; then
    local STATUS="$(git rev-list --left-right --count origin/master..."$(git rev-parse --abbrev-ref HEAD)" \
      | sed -r 's#([[:digit:]]+)\s+([[:digit:]]+)#Atras do origin/master por \1 commit e na frente por \2#')"
    echo "(${STATUS})"
  fi
}
```

### Chamar a função dentro do PS1

O `PS1` no `~/.bashrc` deve ficar algo como

```bash
export PS1='[\u@\h \W] $(_git_status_relation_to_origin_master)\[\e[0m\]\$ '
```

## Efetivando a mudança

Recarregue o terminal ou execute apenas `source ~/.bashrc`.

## Explicação do comando

- O `if` checa se a pasta corrente é um repositório Git
- O `git rev-parse --abbrev-ref HEAD` retorna a branch corrente
- O `sed` formata para que tenhamos a frase informando a diferença
