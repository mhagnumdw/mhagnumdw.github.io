---
author-id: mhagnumdw
categories:
- docker
date: "2020-11-25T16:45:00Z"
feature-img: assets/img/posts/docker-change-internal-date-time/banner.png
image: assets/img/posts/docker-change-internal-date-time/banner.png
tags:
- docker
- time
thumbnail: assets/img/posts/docker-change-internal-date-time/banner.png
title: 'Docker: alterar data e hora sem interferir no host'
---

Alterar a data e hora de um container docker **sem interferir na data e hora do host**.

<!--more-->

- [Tentativas](#tentativas)
  - [Executando `date -s`](#executando-date--s)
  - [Executando `date -s` em um container com `--privileged`](#executando-date--s-em-um-container-com---privileged)
  - [Executando `date -s` em um container com `--cap-add=SYS_TIME`](#executando-date--s-em-um-container-com---cap-addsys_time)
- [Solução definitiva](#solução-definitiva)
- [Como usar isso na prática](#como-usar-isso-na-prática)
- [Dockerfiles para algumas distros](#dockerfiles-para-algumas-distros)
- [Referências](#referências)

## Tentativas

### Executando `date -s`

Ao tentar alterar a data pelo modo tradicional, obtemos um erro:

```console
(host)$ docker run -it --rm --env TZ=America/Fortaleza --privileged debian sh

$ id # checando o usuario corrente
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)

$ date # verificando a data corrente
Wed Nov 25 21:02:49 UTC 2020

$ date -s "2020-12-15 18:00:00" # tentando alterar a data
date: can't set date: Operation not permitted
Tue Dec 15 18:00:00 UTC 2020

$ date # verificando a data corrente (nao foi alterada)
Wed Nov 25 21:02:58 UTC 2020
```

### Executando `date -s` em um container com `--privileged`

Subindo o container com `--privileged`, até conseguimos alterar, **mas também altera no host** (não queremos isso!):

```console
(host)$ docker run -it --rm --env TZ=America/Fortaleza --privileged debian sh

$ date # verificando a data corrente
Wed Nov 25 18:26:23 -03 2020

$ date -s "2020-12-15 18:00:00" # tentando alterar a data
Tue Dec 15 18:00:00 -03 2020

$ date # verificando: foi alterada no container e no host
Tue Dec 15 18:00:07 -03 2020
```

<video muted autoplay controls style="width=:100%;padding: unset;">
    <source src="{{ site.baseurl }}/assets/img/posts/docker-change-internal-date-time/docker-privileged-change-date-time.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

### Executando `date -s` em um container com `--cap-add=SYS_TIME`

Iniciando o container com a opção `--cap-add=SYS_TIME` temos o mesmo ~~problema~~ comportamento acima.

## Solução definitiva

Prover dentro da imagem a lib [libfaketime](https://github.com/wolfcw/libfaketime). A forma mais recomenda é criar um `Dockerfile` para criar a imagem.

Abaixo um exemplo de `Dockerfile` usando o `debian`. Estaremos criando uma imagem a partir da imagem base do `debian` e adicionando apenas a capacidade de poder modificar a data e hora.

```dockerfile
# Builda a lib libfaketime
FROM debian as libfaketime-bin

RUN apt-get update && \
    apt-get install -y git build-essential

RUN git clone https://github.com/wolfcw/libfaketime /libfaketime

WORKDIR /libfaketime

RUN make

RUN make install

#########################

# Builda a imagem debian com a lib libfaketime (imagem definitiva)
FROM debian

COPY --from=libfaketime-bin /usr/local/lib/faketime /usr/local/lib/faketime
```

> 📋 O `Dockerfile` acima possui duas cláusulas `FROM`, mas apenas a última representa a imagem efetiva gerada ao final do comando `docker build`. O nome desse recurso é [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/). É bastante útil quando na construção da imagem existem passos transientes, ex: precisamos apenas do binário, mas não precisamos de toda a stack de build que é necessário para criar o binário.
>
> 📋 O primeiro `FROM` cria uma imagem temporária com a lib `libfaketime` compilada (o binário). O segundo `FROM` (e definitivo) vai copiar **apenas** a lib para a imagem definitiva.

Para testar basta construir a imagem:

```bash
docker build -t faketime-debian -f Dockerfile.debian .
```

> 📋 A única diferença da imagem resultante para a imagem original do `debian`, são dois arquivos **novos**. Podemos checar facilmente com o [dive](https://github.com/wagoodman/dive), executando: `dive faketime-debian`. Essa é uma boa notícia, pois a imagem original praticamente não muda!

Em seguida iniciar um container informando os parâmetros de data:

```bash
docker run -it --rm \
  --env FAKETIME=+15d \
  --env LD_PRELOAD=/usr/local/lib/faketime/libfaketimeMT.so.1 \
  --env DONT_FAKE_MONOTONIC=1 \
  faketime-debian \
  date
```

**A data exibida no console deve ser 15 dias no futuro.**

Observar os parâmetros `FAKETIME`, `LD_PRELOAD` e `DONT_FAKE_MONOTONIC`. Mais opções podem ser consultadas na [página do projeto libfaketime](https://github.com/wolfcw/libfaketime).

## Como usar isso na prática

É comum termos uma imagem e precisarmos desse recurso (ou não, 😃), então dado o exemplo acima, o segundo `FROM` deve ser o `FROM` que você tem na sua imagem atual.

## Dockerfiles para algumas distros

É recomendado compilar (buildar) o `libfaketime` de acordo com a distribuição da sua imagem. Segue abaixo três Dockerfiles para `debian`, `alpine` e `fedora`. No ponto de você trocar o segundo `FROM` para o seu caso.

- [debian]({{ site.baseurl }}/assets/outros/docker-change-internal-date-time/Dockerfile.debian)
- [alpine]({{ site.baseurl }}/assets/outros/docker-change-internal-date-time/Dockerfile.alpine)
- [fedora]({{ site.baseurl }}/assets/outros/docker-change-internal-date-time/Dockerfile.fedora)

## Referências

- <https://github.com/wolfcw/libfaketime>
- <https://github.com/trajano/alpine-libfaketime/blob/master/Dockerfile>
- <https://stackoverflow.com/questions/29556879/is-it-possible-change-date-in-docker-container>
- [LD_PRELOAD](https://www.baeldung.com/linux/ld_preload-trick-what-is)
- <https://docs.docker.com/develop/develop-images/multistage-build/>
