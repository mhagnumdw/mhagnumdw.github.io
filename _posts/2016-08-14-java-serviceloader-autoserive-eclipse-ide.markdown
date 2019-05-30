---
layout: post
title: Java ServiceLoader + @AutoService + Eclipse IDE
date: 2016-08-14 00:23:26 -03:00
categories:
- Java
tags:
- Eclipse
- ServiceLoader
author-id: mhagnumdw
image: "assets/img/posts/java-serviceloader-autoserive-eclipse-ide/java_serviceloader_autoservice_eclipse.png"
feature-img: "assets/img/posts/java-serviceloader-autoserive-eclipse-ide/java_serviceloader_autoservice_eclipse.png"
thumbnail: "assets/img/posts/java-serviceloader-autoserive-eclipse-ide/java_serviceloader_autoservice_eclipse.png"
---

- O que é Java ServiceLoader?
- Como usar o @AutoService?
- Como integrar com o (build) processador de anotações do Eclipse?

* * *

## Java ServiceLoader

É um provedor simples de serviços disponibilizado pela classe java.util.ServiceLoader. As interfaces devem ser registradas como arquivos dentro da pasta META-INF/services/ e as implementações registradas no conteúdo dos arquivos.

<!--more-->

Ver mais:
- [http://www.oracle.com/technetwork/articles/javase/extensible-137159.html](http://www.oracle.com/technetwork/articles/javase/extensible-137159.html)
- [http://docs.oracle.com/javase/6/docs/api/java/util/ServiceLoader.html](http://docs.oracle.com/javase/6/docs/api/java/util/ServiceLoader.html)

* * *

## @AutoService (google/auto)

Gera automaticamente os arquivos necessários para o [java.util.ServiceLoader](http://docs.oracle.com/javase/6/docs/api/java/util/ServiceLoader.html) funcionar.

{% highlight xml %}
<dependency>
    <groupId>com.google.auto.service</groupId>
    <artifactId>auto-service</artifactId>
    <version>1.0-rc2</version>
    <optional>true</optional>
</dependency>
{% endhighlight %}

O código de exemplo abaixo:

{% highlight java %}
package com.demo;
public interface ControllerHandlerFactory {
    //...
}
{% endhighlight %}

{% highlight java %}
package com.demo;
@AutoService(ControllerHandlerFactory.class)
public class DefaultControllerHandlerFactory implements ControllerHandlerFactory {
    //...
}
{% endhighlight %}

{% highlight java %}
package com.demo;
@AutoService(ControllerHandlerFactory.class)
public class GuiceControllerHandlerFactory implements ControllerHandlerFactory {
    //...
}
{% endhighlight %}

**Gera o arquivo:**
/META-INF/services/com.demo.ControllerHandlerFactory

**Contendo as linhas:**
- com.demo.DefaultControllerHandlerFactory
- com.demo.GuiceControllerHandlerFactory

Ver mais: [https://github.com/google/auto/tree/master/service](https://github.com/google/auto/tree/master/service)

* * *

## Eclipse IDE - Integração com o @AutoService

**Realizar o download dos JARs:**
- [https://mvnrepository.com/artifact/com.google.auto.service/auto-service](https://mvnrepository.com/artifact/com.google.auto.service/auto-service)
- [https://mvnrepository.com/artifact/com.google.code.findbugs/jsr305](https://mvnrepository.com/artifact/com.google.code.findbugs/jsr305)
- [https://mvnrepository.com/artifact/com.google.guava/guava](https://mvnrepository.com/artifact/com.google.guava/guava)
- [https://mvnrepository.com/artifact/com.google.auto/auto-common](https://mvnrepository.com/artifact/com.google.auto/auto-common)

_Dica: fazer uso desses jar's a partir do .m2 do maven_

**Seguir os passos:**

1. Botão direito sobre o projeto;
1. Java Compiler > Annotation Processing > Factory Path;
1. Add External JARs...
1. Adicionar os JARs baixados
1. Ok

Aqui o Eclipse passará a gerar, durante o build, os arquivos para o ServiceLoader por meio do @AutoService.
