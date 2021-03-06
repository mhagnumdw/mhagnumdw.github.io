---
layout: post
title: Cygwin SSH Server no Windows 7
date: 2016-08-24 11:47:28 -03:00
categories:
- Cygwin
tags:
- SSH
- Linux
author-id: mhagnumdw
image: "assets/img/posts/cygwin-ssh-server-no-windows-7/cygwin_ssh_server_logo.png"
feature-img: "assets/img/posts/cygwin-ssh-server-no-windows-7/cygwin_ssh_server_logo.png"
thumbnail: "assets/img/posts/cygwin-ssh-server-no-windows-7/cygwin_ssh_server_logo.png"
---

Instalar o Cygwin SSH Server como serviço no Windows.

<!--more-->

## Instalando

Instalar os pacotes abaixo no Cygwin:
_Dica: executar o próprio instalador do Cygwin para instalar os pacotes._

- openssh
- openssl
- cygrunsrv

Após a instalação dos pacotes acima, no terminal do Cygwin, executar:

```shell
ssh-host-config -y
```

O assistente vai criar o usuário `cyg_server` para rodar o serviço SSH e será necessário definir uma senha.

A mensagem abaixo deve ser exibida após a finalização com sucesso:

![Cygwin_SSH_Server_Install]({{ site.baseurl }}/assets/img/posts/cygwin-ssh-server-no-windows-7/cygwin_ssh_server_install.png)

Nesse ponto o serviço SSH está instalado.

## Iniciando o serviço SSH

```shell
cygrunsrv -S sshd
```

## Parando o serviço SSH

```shell
cygrunsrv -E sshd
```

## Outros

- O serviço criado pode ser visualizado no `services.msc` com o nome `CYGWIN sshd`
- Para iniciar o serviço SSH com outro usuário, utilizar o parâmetro `-u` no comando `ssh-host-config`
