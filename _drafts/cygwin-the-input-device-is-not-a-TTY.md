---
layout: post
title:  "Cygwin: the input device is not a TTY"
date: 2019-10-21 08:00:00 -03:00
categories:
- Cygwin
tags:
- cygwin
- tty
author-id: mhagnumdw
image: "assets/img/posts/cygwin-input-not-tty/xxx.png"
feature-img: "assets/img/posts/cygwin-input-not-tty/xxx.png"
thumbnail: "assets/img/posts/cygwin-input-not-tty/xxx.png"
---


Em alguns casos ao rodar programas que interagem com o terminal no Cygwin podemos ter uma mensagem de erro parecida com essa

"the input device is not a TTY"

ou com essa se tiver tentando fazer um 'docker login'

"Error: Cannot perform an interactive login from a non TTY device"


A solução é usar o 'winpty'. O binário pode ser obtido aqui: https://github.com/rprichard/winpty/releases

Descomprimir o arquivo tar.gz. Na pasta gerada tem uma pasta bin e dentro 4 arquivos:
- winpty.dll
- winpty.exe
- winpty-agent.exe
- winpty-debugserver.exe

Copiar todos para dentro de C:\cygwin64\usr\local\bin

Agora basta executar o programa precedido de winpty, ex:

- winpty docker login --username eu registry-git.meudominio.com
- winpty docker run --rm -it alpine sh


Para saber:
O git-bash (MINGW64) já traz o winpty.

Referências:
- https://stackoverflow.com/questions/43248455/gitlab-runner-local-build-login-from-non-tty-device
- https://github.com/rprichard/winpty/issues/139
