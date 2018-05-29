---
layout: post
title: Cygwin SSH Server no Windows 7
date: 2016-08-24 11:47:28 -03:00
categories:
- Cygwin
tags:
- SSH
- Linux
author: mhagnumdw
feature-img: "assets/test_v3.png"
thumbnail: "assets/test_v3.png"
---

Instalar o Cygwin SSH Server como serviço no Windows.

## Instalando

Instalar os pacotes abaixo no Cygwin:  
_Dica: executar o próprio instalador do Cygwin para instalar os pacotes._

- openssh
- openssl
- cygrunsrv

Após a instalação dos pacotes acima, no terminal do Cygwin, executar:

{% highlight shell %}
ssh-host-config -y
{% endhighlight %}

O assistente vai criar o usuário `cyg_server` para rodar o serviço SSH e será necessário definir uma senha.

A mensagem abaixo deve ser exibida após a finalização com sucesso:

![Cygwin_SSH_Server_Install]({{ site.baseurl }}/assets/cygwin_ssh_server_install.png)

Nesse ponto o serviço SSH está instalado.

## Iniciando o serviço SSH

{% highlight shell %}
cygrunsrv -S sshd
{% endhighlight %}

## Parando o serviço SSH

{% highlight shell %}
cygrunsrv -E sshd
{% endhighlight %}

## Outros

- O serviço criado pode ser visualizado no `services.msc` com o nome `CYGWIN sshd`;
- Para iniciar o serviço SSH com outro usuário, utilizar o parâmetro `-u` no comando `ssh-host-config`.