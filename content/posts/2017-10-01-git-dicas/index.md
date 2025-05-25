---
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"

categories:
- Git
date: "2017-10-01T13:32:22Z"

image: assets/img/posts/git-dicas/git-goodness_v2.png
tags:
- git
- git-dicas

title: 'git: dicas'
---

Dicas de git (snippets)

<!--more-->

Índice

- [Criar uma branch vazia com nome "maven-repo"](#criar-uma-branch-vazia-com-nome-maven-repo)
- [Visualizar exclusivamente os commits feitos em uma branch em relação ao master](#visualizar-exclusivamente-os-commits-feitos-em-uma-branch-em-relação-ao-master)
- [Diferença de commits entre duas branches](#diferença-de-commits-entre-duas-branches)
- [Merge entre branches de repositórios diferentes](#merge-entre-branches-de-repositórios-diferentes)
- [Listar os autores em um projeto](#listar-os-autores-em-um-projeto)
- [Adicionar ao stash sem remover do workspace](#adicionar-ao-stash-sem-remover-do-workspace)
- [Obter um arquivo específico de um stash](#obter-um-arquivo-específico-de-um-stash)
- [Deletar branch local e remota](#deletar-branch-local-e-remota)
- [Buscar no conteúdo dos arquivos em todo o histórico](#buscar-no-conteúdo-dos-arquivos-em-todo-o-histórico)
- [Exibir o autor no `git rebase -i`](#exibir-o-autor-no-git-rebase--i)
- [Exibir o autor no estilo `git log --oneline`](#exibir-o-autor-no-estilo-git-log---oneline)
- [Checkout por data e hora](#checkout-por-data-e-hora)
- [`git log` por data e hora](#git-log-por-data-e-hora)

### Criar uma branch vazia com nome "maven-repo"

```shell
git clone https://github.com/mhagnumdw/pippo.git
cd pippo/
git checkout --orphan maven-repo
git rm -rf .
git commit --allow-empty -m "Apagando todos os arquivos da branch para servir de repositório maven"
git push origin maven-repo
```

* * *

### Visualizar exclusivamente os commits feitos em uma branch em relação ao master

```shell
git branch
git checkout feature
git cherry -v master
```

* * *

### Diferença de commits entre duas branches

```shell
git log --oneline origin/master..HEAD
# ou
diff -y <(git log --oneline -10) <(git log --oneline origin/master -10)
```

* * *

### Merge entre branches de repositórios diferentes

_Se você quiser fazer um merge de project-a para project-b:_

```shell
cd path/to/project-b
git remote add project-a path/to/project-a
git fetch project-a
git merge --allow-unrelated-histories project-a/master
git remote remove project-a
```

[https://stackoverflow.com/questions/1425892/how-do-you-merge-two-git-repositories#answer-10548919](https://stackoverflow.com/questions/1425892/how-do-you-merge-two-git-repositories#answer-10548919)

* * *

### Listar os autores em um projeto

```shell
git shortlog --summary --numbered --email
```

* * *

### Adicionar ao stash sem remover do workspace

```shell
git stash push --include-untracked \
  --message 'Melhoria do autocomplete usando rxjs - v001' && \
  git stash apply
```

* * *

### Obter um arquivo específico de um stash

```shell
# listar stashs
git stash list
# listar os arquivos de um stash
git stash show stash@{0}
# ver o conteúdo do arquivo
git show stash@{0}:src/main/java/br/log/AlertsManager.java
# obter o arquivo para o workspace
git checkout stash@{0} -- src/main/java/br/log/AlertsManager.java
```

* * *

### Deletar branch local e remota

```shell
# deletar branch remota
git push --delete origin angular_gitlab_build_problem
# deletar branch local
git branch -d angular_gitlab_build_problem
```

* * *

### Buscar no conteúdo dos arquivos em todo o histórico

Buscar um texto em **todos os arquivos** em **todo o histórico**.

```shell
# achar o bloco de código sem se importar com o arquivo
git log --patch | grep -P -B 20 -A 3 '(addShutdownHook|threadToDeleteOnExit)'

# achar os commits que possuem os arquivos com o termo buscado
git log -SaddShutdownHook --name-status
git log -GaddShutdownHook --name-status
# após o S e G deve ser o termo buscado, que pode estar entre aspas simples
# a opção G aceita regex
```

* * *

### Exibir o autor no `git rebase -i`

```shell
git config --add rebase.instructionFormat "(%an <%ae>) %s"

# ou global
git config --global rebase.instructionFormat "(%an <%ae>) %s"
```

* * *

### Exibir o autor no estilo `git log --oneline`

```shell
git log --pretty=format:"%C(yellow)%h %C(auto,magenta)%>(12)[%ad] %Cgreen%<(7)%aN (%aE)%Cred%d %Creset%s" --date=format:"%d/%m/%Y %H:%M:%S"
```

Criar um alias para evitar toda essa linha:

```shell
# definir globalmente um alias chamado logp
git config --global alias.logp 'log --pretty=format:"%C(yellow)%h %C(auto,magenta)%>(12)[%ad] %Cgreen%<(7)%aN (%aE)%Cred%d %Creset%s" --date=format:"%d/%m/%Y %H:%M:%S"'

# agora basta
git logp
git logp -5
git logp origin/master..
```

* * *

### Checkout por data e hora

```shell
git checkout `git rev-list -n 1 --before="2021-03-24 17:00" master`
```

Observação: é possível ver por aí, pesquisando na net, a solução `git checkout master@{2021-03-24 17:00}`, **não vai funcionar bem**, porque ele usa o reflog (que expira depois de algum tempo).

ref: <http://web.archive.org/web/20170303015800/http://bramschoenmakers.nl/en/node/645>

* * *

### `git log` por data e hora

```shell
# ex1
git log --oneline --after="2020-01-01 00:00"

# ex2
git log --oneline --before="2020-01-01 00:00"
```

* * *
