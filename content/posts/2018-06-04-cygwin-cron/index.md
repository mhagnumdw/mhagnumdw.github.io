---
author-id: mhagnumdw
categories:
- Cygwin
date: "2018-06-04T20:03:00Z"
feature-img: assets/img/posts/cygwin-cron/cygwin-cron.png
image: assets/img/posts/cygwin-cron/cygwin-cron.png
tags:
- cron
thumbnail: assets/img/posts/cygwin-cron/cygwin-cron.png
title: Cygwin Cron
---

Instalar o cron no Cygwin como serviço no Windows.

<!--more-->

## Passos

- Instalar o `cygrunsrv` no Cygwin

- Iniciar o Cygwin como administrador do Windows

- Registrar o serviço

```shell
cygrunsrv -I CYGWIN-cron -u SEU_USUARIO_AQUI -p /usr/sbin/cron -a "-n"
```

- Ver detalhes do serviço

```shell
cygrunsrv --list -V
```

> 📋 A saída do comando acima deve mostrar que o serviço está parado

- Iniciando o serviço

```shell
cygrunsrv --start CYGWIN-cron
```

- Ver detalhes do serviço

```shell
cygrunsrv --list -V
```

> 📋 A saída do comando acima deve mostrar que o serviço está em execução

- Verificar que o serviço está cadastrado no Windows

```shell
cmd /c 'services.msc'
```

_Buscar pelo serviço_ `CYGWIN-cron`

- Agora é só registrar alguma execução no cron

```shell
# em um sessao do cygwin com seu usuario definido no inicio
crontab -e
# ou, se você quiser especificar o editor
env VISUAL="vim" crontab -e
```
