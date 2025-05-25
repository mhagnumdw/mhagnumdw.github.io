---
author-id: mhagnumdw
categories:
- Linux
date: "2018-06-05T14:39:00Z"
feature-img: assets/img/posts/fedora-28-dnf-instalar-pacote-antigo-e-desativar-atualizacao/fedora-dnf-package-old-disable.png
image: assets/img/posts/fedora-28-dnf-instalar-pacote-antigo-e-desativar-atualizacao/fedora-dnf-package-old-disable.png
tags:
- Fedora
- dnf
- yum
thumbnail: assets/img/posts/fedora-28-dnf-instalar-pacote-antigo-e-desativar-atualizacao/fedora-dnf-package-old-disable.png
title: 'Fedora 28 + DNF: instalar pacote antigo e desativar atualização'
---

Instalar uma versão específica de um pacote e desativar sua atualização ao executar `dnf update`.

<!--more-->

Para o exemplo abaixo será utilizado o pacote do `subversion`.

#### Listar as versões do pacote

```shell
dnf --showduplicates list subversion
```

#### Instalar a versão desejada

```shell
sudo dnf install subversion-1.9.7-6.fc28
```

Especificar a versão desejada concatenado o nome do pacote com a versão.

#### Excluir o pacote da atualização do DNF

Editar o arquivo `sudo vim /etc/dnf/dnf.conf` e adicionar a linha

```text
exclude=subversion subversion-javahl subversion-lib
```

A partir de então o `dnf update` não irá atualizar os pacotes definidos acima.
