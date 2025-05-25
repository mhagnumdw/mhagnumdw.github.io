---
author-id: mhagnumdw
categories:
- Cygwin
date: "2016-08-24T11:47:28Z"
feature-img: assets/img/posts/cygwin-ssh-server-no-windows-7/cygwin_ssh_server_logo.png
image: assets/img/posts/cygwin-ssh-server-no-windows-7/cygwin_ssh_server_logo.png
tags:
- SSH
- Linux
thumbnail: assets/img/posts/cygwin-ssh-server-no-windows-7/cygwin_ssh_server_logo.png
title: Cygwin SSH Server no Windows 7
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
