---
layout: post
title: 'Diário de Bordo #001 - Demora para se conectar via SSH'
date: 2020-05-16 12:22:00 -03:00
categories:
- Diário de Bordo
tags:
- docker
- ssh
- dns
author-id: mhagnumdw
image: "assets/img/cockpit-banner.jpg"
feature-img: "assets/img/cockpit-banner.jpg"
thumbnail: "assets/img/cockpit-banner-900.jpg"
---

Teste automatizado, Docker, SVN Subversion container, demora de aproximadamente 10 segundos para conectar via SSH ⏩ Troubleshooting

<!--more-->

## História

{% include about_diario_de_bordo.markdown %}

Tenho alguns testes automatizados escritos em Java que sobem um container do SVN e se conectam ao container via SSH utilizando chave púlica/privada.

A biblioteca Java utilizada para fazer acesso SSH é a [jsch](http://www.jcraft.com/jsch/) na versão [0.1.55](https://mvnrepository.com/artifact/com.jcraft/jsch/0.1.55). Inicialmente pensei que o problema fosse nela, inclusive precisei escrever uma classe de log para ela seguindo [esse modelo](http://www.jcraft.com/jsch/examples/Logger.java.html) (estranho!), onde é possível vincular ao sistema de log da minha aplicação.

Agora com o log do jsch e debugando o código dele, percebi que o código ficava esperando um [InputStream](https://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html) de um socket no método `com.jcraft.jsch.IO.getByte(byte[], int, int)`, então comecei a pensar que o problema estava no servidor de SSH no container. Antes disso pesquisei sobre lentidão na conexão SSH com jsch, na esperança de achar algum parâmetro mágico e pouco achei e nada serviu.

Para elimintar de vez que o problema poderia estar no jsch, resolvi me conectar ao container via linha de comando (porque não pensei nisso antes!). Ao me conectar, exatamente com a linha `ssh -i chave-privada -p 2220 user@127.0.0.1`, assim como fazia meu teste automatizado, também obtive o delay de aproximadamente 10 segundos para concluir a conexão.

> **NOTA:** o IP 127.0.0.1 é porque o container subiu fazendo bind das portas para o meu host.

Agora eu já sabia que a lentidão era alguma coisa no servidor, no daemon do SSH, ou ao menos eu tinha 99% de certeza. Ativei então o modo verboso no cliente, me conectando assim: `ssh -vvvvv -i chave-privada -p 2220 user@127.0.0.1`. No log percebi que a demora toda era na linha `debug3: send packet: type 50`. Mas o que é isso? Google!

Então pesquisei exatamente por [ssh slow "send packet: type 50"](https://www.google.com/search?q=ssh%20slow%20%22send%20packet:%20type%2050%22) e cheguei no link <https://tanelpoder.com/posts/troubleshooting-linux-ssh-logon-delay-always-takes-10-seconds/>.

> **NOTA:** esse link é **MUITO** bom. 📋

Resumo do link: o chapa explica de forma detalhada e clara como ele descobriu o problema, inclusive fazendo um [strace](https://en.wikipedia.org/wiki/Strace) do processo do daemon do SSH (sshd). Fica claro que o problema era o tempo para executar um lookup reverso do DNS, ou seja, a partir do IP obter o nome.

💭 Pensei: será isso? 🏃‍♂️ Entrei no container `docker exec -it svn bash` e executei `nslookup IP_DO_HOST` e o comando demorou 10 segundos (_nem cheguei a fazer o strace_)!  Como eu não precisava disso para os meus testes, agora bastava descobrir como desativar esse comportamento no servidor SSH. Google!

Bastou então adicionar os parâmetros abaixo ao arquivo `/etc/ssh/sshd_config` ...

- `UseDNS no`
- `UsePAM no`

... e reiniciar o sshd com `service sshd restart` já que o container é Ubuntu.

> **NOTA:** tenho dúvidas se o `UsePAM no` é necessário, embora eu saiba o que ele faz.

Agora, tanto via linha de comando como pelos meus testes automatizados, a demora para se conectar foi embora.

Até a próxima!

## Referências

- <https://tanelpoder.com/posts/troubleshooting-linux-ssh-logon-delay-always-takes-10-seconds/>
- <https://unix.stackexchange.com/questions/298698/ssh-very-slow-connection#298797>
- <https://wiki.alpinelinux.org/wiki/Setting_up_a_ssh-server#Fine_tuning>
