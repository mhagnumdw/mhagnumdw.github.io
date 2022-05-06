---
layout: post
title: 'Log4j1 - MDC não funciona no Java 9+'
date: 2022-05-06 16:18:00 -03:00
categories:
- log4j1
tags:
- log4j
- mdc
- java
author-id: mhagnumdw
image: "assets/img/posts/log4j1-mdc-problem-java17/banner.png"
feature-img: "assets/img/posts/log4j1-mdc-problem-java17/banner.png"
thumbnail: "assets/img/posts/log4j1-mdc-problem-java17/banner.png"
---

O MDC do Log4j1 não aparece no log no Java 9+.

<!--more-->

O Log4j1 precisa saber a versão do Java para decidir se a feature de MDC (Mapped Diagnostic Context) pode ser utilziada ou não. No Java v1 não era possível usar essa feature.

O Java a partir da versão 9 [mudou a lei de formação do número da versão](http://openjdk.java.net/jeps/223) e isso fez a classe [org.apache.log4j.helpers.Loader](https://github.com/apache/logging-log4j1/blob/v1_2_17/src/main/java/org/apache/log4j/helpers/Loader.java#L42-L50) do Log4j1 detectar erroneamente a versão do Java.

Uma JVM com versão `17-ea` que é versão 17, é detectada como versão 1. Exemplo de jvm que tem versão 17-ea: `docker run -it --rm openjdk:17-jdk-alpine java -version`.

## Algumas soluções

- Atualizar para o Log4j2
- Utilizar o [Logback](https://logback.qos.ch/), que é outro provedor de log que dá suporte ao MDC
- Sobrescrever fisicamente a classe `org.apache.log4j.helpers.Loader` se sua versão do Java for posterior a 1 **(vamos focar nessa)**

## Solução: sobrescrever fisicamente a classe `org.apache.log4j.helpers.Loader`

Se por qualquer motivo você não puder trocar a versão do log4j1 ou mesmo mudar para outro provedor de log, é possível substituir a classe `org.apache.log4j.helpers.Loader` colocando-a no seu projeto no mesmo pacote de origem dela.

![loader-dentro-do-projeto]({{ site.baseurl }}/assets/img/posts/log4j1-mdc-problem-java17/loader-dentro-do-projeto.png)

Agora faça as modificações no arquivo `Loader.java` conforme instruções abaixo:

- Comente o código conforme a imagem:

![diff1]({{ site.baseurl }}/assets/img/posts/log4j1-mdc-problem-java17/loader-diff-1.png)

- O método `isJava1()` deve sempre retornar `true`
- Os locais que referenciam a variável `java1` devem passar a chamar o método `isJava1()`

Agora o MDC vai aparecer no log.

## Referências

- <http://openjdk.java.net/jeps/223>
- <https://blogs.apache.org/logging/entry/moving_on_to_log4j_2>
- <https://stackoverflow.com/questions/69822992/why-log4j-stop-running-mdc-logic-if-java-version-is-11-11-1x-etc/72147021>
- <https://stackoverflow.com/questions/54254773/log4j-mdc-loader-java1-returns-true-on-jdk11>
