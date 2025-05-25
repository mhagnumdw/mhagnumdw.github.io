---
title: Jenkins recente buildando com JDK antiga
date: "2019-05-29T10:06:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
resources:
- name: "featured-image"
  src: "build-with-old-jdk-in-recent-jenkins-banner.jpeg"
categories: ["jenkins"]
tags: ["jenkins"]
---

Por exemplo, a partir de tal versão do Jenkins a JDK 8 é necessária e isso pode ser um problema para buildar projetos que requerem uma JDK mais antiga.

<!--more-->

## Motivação (caso de uso)

O Hibernate 3.6 necessita da JDK 5 para buildar corretamente. Mesmo adicionando no Jenkins a JDK 5, o maven 3.1.1 que é a última versão compatível para essa JDK e configurando o job para fazer uso dessas versões, teremos o problema: `Exception in thread "main" java.lang.UnsupportedClassVersionError: Bad version number in .class file`

## Ambiente em questão

- Jenkins 2.32.3 com `JDK 8`
- Versão do Maven no Jenkins: 3.2.1
- Projeto para buildar: Hibernate 3.6 com `JDK 5`

## Configurando o toolchain / maven-toolchains-plugin no Jenkins e no projeto

> What is Toolchains? The Maven Toolchains provide a way for plugins to discover what JDK (or other tools) are to be used during the build ...

### Jenkins (configuração global)

- Instalar o plugin: [Config File Provider Plugin](https://wiki.jenkins.io/display/JENKINS/Config+File+Provider+Plugin)
- Criar configuração do toolchain: `Jenkins > Manage Jenkins > Managed files > Add a new Config > Maven toolchains.xml > Submit`
- Adicionar as informações abaixo

> **Name:**
> Hibernate-Toolchains
>
> **Content:**
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <toolchains>
>   <toolchain>
>      <type>jdk</type>
>      <provides>
>          <version>1.5</version>
>          <vendor>oracle</vendor>
>      </provides>
>      <configuration>
>         <jdkHome>/usr/java/jdk1.5.0_22</jdkHome>
>      </configuration>
>   </toolchain>
> </toolchains>
> ```
>
![Maven toolchains.xml]({{ "assets/img/posts/build-with-old-jdk-in-recent-jenkins/managed-files-config.png" | relative_url }})

- Clicar em Submit

Então será redirecionado para todos os arquivos de configuração.

> A configuração acima apenas indica onde está o binário da jdk da Oracle versão 1.5 para um projeto que esteja configurado com o plugin `maven-toolchains-plugin`

### Jenkins (configuração no job)

- Selecione a JDK
- Configure a URL do seu SCM (SVN, git ...)
- E a parte foco desse post, configurar a seção `Build Environment`:
  1. Marcar o checkbox `Provide Configuration files`
  1. E preencher:
     1. File: `Hibernate-Toolchains`
     1. Variable: `TOOLCHAIN`
- Na seção `Build`, em `Goals and options` referenciar a configuração assim: `-t $TOOLCHAIN`
  - Exemplo completo de Goals: `-t $TOOLCHAIN -B -V clean package verify -DskipTests -Dmaven.test.skip=true -DdisableDistribution=true -Djdk16_home=/usr/lib/jvm/java-1.7.0-oracle-1.7.0.75-1jpp.2.el7.x86_64`

![Build Environment]({{ "assets/img/posts/build-with-old-jdk-in-recent-jenkins/build-enviroment.png" | relative_url }})

![Build - Goals and options]({{ "assets/img/posts/build-with-old-jdk-in-recent-jenkins/build-goals-and-options.png" | relative_url }})

### Configurar o projeto

Configuração no pom.xml do projeto (snippet)

```xml
<build>
    <plugins>
        <!-- indica que deve compilar com jdk 5 da oracle -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-toolchains-plugin</artifactId>
            <version>1.1</version>
            <executions>
                <execution>
                    <goals>
                        <goal>toolchain</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <toolchains>
                    <jdk>
                        <version>1.5</version>
                        <vendor>oracle</vendor>
                    </jdk>
                </toolchains>
            </configuration>
        </plugin>
    </plugins>

    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-toolchains-plugin</artifactId>
                <version>1.1</version>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

## É isso

A partir de então, mesmo que o maven do Jenkins seja invocado com a jdk 8, o build do Hibernate vai acontecer com a jdk 5.

## Referências

- <https://stackoverflow.com/questions/46276236/jenkins-error-when-i-try-to-use-an-older-jdk-for-a-specific-maven-project?rq=1>
- <http://maven.apache.org/plugins/maven-toolchains-plugin/toolchains/jdk.html>
- <https://support.cloudbees.com/hc/en-us/articles/115003910392-How-to-config-a-Maven-Toolchain>
- <https://maven.apache.org/docs/history.html>
