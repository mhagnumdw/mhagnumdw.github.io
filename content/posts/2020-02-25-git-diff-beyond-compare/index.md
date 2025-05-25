---
title: Git Diff com Beyond Compare
date: "2020-02-25T09:34:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
resources:
- name: "featured-image"
  src: "git-beyond-compare-banner.png"
categories: ["git"]
tags: ["git", "diff"]
---

Usar o Beyond Compare como ferramenta de **diff** para o Git.

<!--more-->

## Cygwin e MinGW (Windows)

```shell
git config --global diff.tool bc3
git config --global difftool.bc3.trustExitCode true

# Para Cygwin
git config --global difftool.bc3.cmd '"/cygdrive/c/Program Files/Beyond Compare 4/BCompare.exe" `cygpath -w "$LOCAL"` `cygpath -w "$REMOTE"`'

# Para MinGW
git config --global difftool.bc3.cmd '"c:/program files/beyond compare 4/bcomp.exe" "$LOCAL" "$REMOTE"'
```

Para desativar o prompt "Launch ... [Y/n]?"

```shell
git config --global difftool.prompt false
```

Exemplos

```shell
git difftool master..minha-branch -- frontend/user/user-list.component.html
git difftool filename.ext
```

> Versões aqui testadas:
>
> - Beyond Compare 4
> - Git 2.21.0 / 2.21.0.windows.1

## Linux

Em breve

## Referências

- <https://www.scootersoftware.com/support.php?zz=kb_vcs>
