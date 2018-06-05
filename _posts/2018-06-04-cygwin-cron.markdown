---
layout: post
title: 'Cygwin Cron'
date: 2018-06-04 20:03:00 -03:00
categories:
- Cygwin
tags:
- cron
author: mhagnumdw
feature-img: "assets/cygwin-cron.png"
thumbnail: "assets/cygwin-cron.png"
---

Instalar o cron no Cygwin como serviço no Windows.

<!--more-->

## Passos

- Instalar o `cygrunsrv` no Cygwin

- Iniciar o Cygwin como administrador do Windows

- Registrar o serviço
{% highlight shell %}
cygrunsrv -I CYGWIN-cron -u SEU_USUARIO_AQUI -p /usr/sbin/cron -a "-n"
{% endhighlight %}

- Ver detalhes do serviço
{% highlight shell %}
cygrunsrv --list -V
{% endhighlight %}
_A saída do comando acima deve mostrar que o serviço está parado_

- Iniciando o serviço
{% highlight shell %}
cygrunsrv --start CYGWIN-cron
{% endhighlight %}

- Ver detalhes do serviço
{% highlight shell %}
cygrunsrv --list -V
{% endhighlight %}
_A saída do comando acima deve mostrar que o serviço está em execução_

- Verificar que o serviço está cadastrado no Windows
{% highlight shell %}
cmd /c 'services.msc'
{% endhighlight %}
_Buscar pelo serviço_ `CYGWIN-cron`

- Agora é só registrar alguma execução no cron
{% highlight shell %}
# em um sessao do cygwin com seu usuario definido no inicio
crontab -e
# ou, se você quiser especificar o editor
env VISUAL="vim" crontab -e
{% endhighlight %}
