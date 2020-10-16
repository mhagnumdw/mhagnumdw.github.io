---
layout: post
title: GitLab/Git + Trello = Integração (dados do commit)
date: 2016-09-01 16:58:30 -03:00
categories:
- Git
- Linux
tags:
- Cygwin
- GitLab
- Trello
author-id: mhagnumdw
image: "assets/img/posts/gitlabgit-trello-integracao-dados-do-commit/git-gitlab-trello-integracao-banner_v2.png"
feature-img: "assets/img/posts/gitlabgit-trello-integracao-dados-do-commit/git-gitlab-trello-integracao-banner_v2.png"
thumbnail: "assets/img/posts/gitlabgit-trello-integracao-dados-do-commit/git-gitlab-trello-integracao-banner_v2.png"
---

Integrar GitLab e Trello no post-commit (client side) ou post-receive (server side).

<!--more-->

Commit e push efetivados terão seus dados disponibilizados nos cards do Trello.

## Instalar NodeJS

### Fedora

```shell
sudo dnf --nogpg install node
```

### Cygwin

Realizar download para Windows e instalar: [http://nodejs.org/](http://nodejs.org/)

## Instalar pacotes do NodeJS necessários para o hook

### Fedora

```shell
sudo npm install -g node-trello
```

![npm install node-trello]({{ site.baseurl }}/assets/img/posts/gitlabgit-trello-integracao-dados-do-commit/gitlabgit-trello-integracao-dados-do-commit-004.png)

```shell
sudo npm install -g colors
```

![npm install colors]({{ site.baseurl }}/assets/img/posts/gitlabgit-trello-integracao-dados-do-commit/gitlabgit-trello-integracao-dados-do-commit-005.png)

### Cygwin

```shell
npm install -g node-trello
npm install -g colors
```

## Obter a chave do Trello

Acessar: [https://trello.com/1/appKey/generate](https://trello.com/1/appKey/generate)

## Obter o token para o hook acessar o Trello

`CHAVE_TRELLO` - valor obtido no passo anterior

Acessar

```text
https://trello.com/1/connect?key=**CHAVE_TRELLO**&name=git-hook&expiration=never&response_type=token&scope=read,write
```

![trello token]({{ site.baseurl }}/assets/img/posts/gitlabgit-trello-integracao-dados-do-commit/gitlabgit-trello-integracao-dados-do-commit-006.png)

## Obter o ID do quadro do Trello

- Abrir o quadro no trello
- Colocar .json no final da URL
- Enter!

No início do json vai ter o ID do quadro:

```text
{"id":"**57beXXXXXXXXXXXXXXXXXXXX**","name":"TechThingsCool", .....
```

## Exibir os IDs dos cards

- Abrir o quadro no trello
- Colocar o Javascript abaixo na barra de endereço do browser
- Enter!

```javascript
javascript:!function(){var o=$(".card-short-id");o.each(function(){$(this).text($(this).text().replace("","").replace("","").replace("N.º ", ""))});o.hasClass("hide")?o.removeClass("hide").css({"font-weight":"normal","font-size":".9em","margin-right":"5px",padding:"2.3px 6px",background:$("body").css("background-color"),"border-radius":"10px",color:"yellow"}):o.addClass("hide")}();
```

## Hook post-commit ou post-receive?

Se desejar que a atualização dos cards do Trello seja:

- a cada commit E no lado cliente, use o hook `post-commit`
- após o push E no lado servidor, use o hook `post-receive`

Abaixo as duas abordagens.

## Hook post-commit (git client com cygwin/Fedora)

Desse modo a cada commit o Trello será atualizado. A execução ocorre no lado cliente.

### Obter os arquivos `post-commit` e `post-to-trello-post-commit` em

[https://drive.google.com/open?id=0B80EagoWEV2xdFFxbFg3aXE1RHM](https://drive.google.com/open?id=0B80EagoWEV2xdFFxbFg3aXE1RHM)

- Colocar na pasta `.git/hooks` do projeto
- Os arquivos devem ser executáveis: `chmod +x`
- No script `post-to-trello-post-commit` substituir os valores: `CHAVE_TRELLO`, `TOKEN_TRELLO`, `GIT_REPO_LINK`, `TRELLO_QUADRO_ID`

### Testando a integração

Efetuar um commit com a mensagem no formato: `#CARD_ID Minha mensagem`

Exemplo

```shell
git commit -m "#7 Teste integração: Trello e GitLab. post-commit."
```

![resultado post-commit]({{ site.baseurl }}/assets/img/posts/gitlabgit-trello-integracao-dados-do-commit/gitlabgit-trello-integracao-dados-do-commit-008.png)

![card trello comentado]({{ site.baseurl }}/assets/img/posts/gitlabgit-trello-integracao-dados-do-commit/gitlabgit-trello-integracao-dados-do-commit-007.png)

Obs: pode ser necessário ajustar `export NODE_PATH` do script/hook `post-commit`, devendo apontar para o local onde se encontra os módulos do NodeJS.

## Hook post-receive (git server no Fedora)

Desse modo o Trello será atualizado apenas no push. Cada commit no push pode apontar para cards diferentes. A execução ocorre no lado servidor.

### Obter os arquivos `post-receive` e `post-to-trello-post-receive` em

[https://drive.google.com/open?id=0B80EagoWEV2xM1N4OV9fSVEteUU](https://drive.google.com/open?id=0B80EagoWEV2xM1N4OV9fSVEteUU)

- Colocar na pasta `/var/opt/gitlab/git-data/repositories/<group>/<project>.git/custom_hooks/`
- Os arquivos devem ser executáveis: `chmod +x`
- No script `post-to-trello-post-receive` substituir os valores: `CHAVE_TRELLO`, `TOKEN_TRELLO`, `GIT_REPO_LINK`, `TRELLO_QUADRO_ID`

### Testando a integração

- Efetuar mais de um commit com a mensagem no formato: `#CARD_ID Minha mensagem`
- Realizar o push
- Observar o log e o Trello (ver imagens acima!)

## Referências

- [http://inteist.com/git-hooks-for-trello/](http://inteist.com/git-hooks-for-trello/)
- [http://joelherber.com/connect-gitlab-to-trello/](http://joelherber.com/connect-gitlab-to-trello/)
- [http://docs.gitlab.com/ce/administration/custom_hooks.html](http://docs.gitlab.com/ce/administration/custom_hooks.html)
- [https://git-scm.com/docs/githooks](https://git-scm.com/docs/githooks)
- [https://nodejs.org/api/child_process.html](https://nodejs.org/api/child_process.html)
