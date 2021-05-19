---
layout: post
title: 'Maven: prevenir compilação desnecessária'
date: 2021-05-19 18:20:00 -03:00
categories:
- maven
- pipeline
tags:
- maven
- pipeline
author-id: mhagnumdw
image: "assets/img/posts/maven-prevenir-compilacao-desnecessaria/banner.png"
feature-img: "assets/img/posts/maven-prevenir-compilacao-desnecessaria/banner.png"
thumbnail: "assets/img/posts/maven-prevenir-compilacao-desnecessaria/banner.png"
---

O [Maven](https://maven.apache.org/) faz uso do plugin [maven-compiler-plugin](http://maven.apache.org/plugins/maven-compiler-plugin/) para compilar os `.java` em `.class`. O plugin já é otimizado para só efetuar uma nova compilação se necessário. Em alguns casos o plugin pode compilar novamente o source e não desejamos esse comportamento, vamos ver como contornar isso.

<!--more-->

> 📋 Esse post se aplica mais a [Pipelines](https://en.wikipedia.org/wiki/Pipeline_(software)), mas recomendo você continuar a leitura 😃

## Verificando que o plugin, por padrão, não compila de forma desnecessária

Em um projeto Maven, executar:

```bash
$ mvn clean compile
[INFO] --- maven-compiler-plugin:2.3.1:compile (default-compile) @ myproject --
[INFO] Compiling 1062 source files to /home/mhagnumdw/myproject/target/classes
```

> É possível ver que foram compilados 1062 arquivos.

Agora vamos empacotar:

```bash
$ mvn package
[INFO] --- maven-compiler-plugin:2.3.1:compile (default-compile) @ myproject --
[INFO] Nothing to compile - all classes are up to date
```

E é possível ver que nada foi compilado, o maven aproveitou o que já existe na pasta `target`.

## Como o Maven sabe o que precisa ou não compilar?

Se o `.class` de um `.java` não existir na pasta `target` **ou se a data de modificação do `.class` for anterior a do `.java`**, o plugin vai disparar a compilação desses sources.

## Caso de uso em que o Maven acaba re-compilando sem necessidade

Em [Pipelines](https://en.wikipedia.org/wiki/Pipeline_(software)), como os do [GitLab](https://docs.gitlab.com/ce/ci/pipelines/) por exemplo, é comum dividirmos as fases envolvidas no processo de build (`compile`, `test`, `package` etc) em estágios distintos. Assim existe uma fase que compila o código gerando os `.class`, e uma fase seguinte pega esse compilado para empacotar e gerar o `.jar`, por exemplo.

Então imagine um pipeline, para simplificar, com apenas dois estágios, chamados `compilação` e `empacotamento` e que executam exatamente nessa ordem.

- O estágio `compilação` tem as seguintes etapas:
  - obtém o código do repositório Git (git clone)
  - executa um `mvn clean compile`
  - guarda a pasta `target` para o estágio `empacotamento`

- O estágio `empacotamento` tem as seguintes etapas:
  - obtém o código do repositório Git (git clone) `[1]`
  - obtém a pasta `target` do estágio `compilação` `[2]`
  - executa um `mvn package` `[3]`
  - guarda a pasta `target` para um estágio posterior

O `mvn packge` `[3]` acaba compilando o código novamente, pois ao obter o código do repositório Git `[1]` a data de modicação dos arquivos fica sendo a do instante do `git clone` e essa data é posterior a data dos `.class` que estão no `target` `[2]`, desse modo o Maven acredita que os `.java` sofreram modificação e executa a compilação (novamente).

> 📋 O comando `stat nome-do-arquivo`  exibe informações de data do arquivo, entre outras coisas.

## Solução

### Solução 1: executar tudo em um único estágio

Se o `mvn clean compile` e `mvn package` estiverem em um mesmo estágio, o source e target serão os mesmos e não haverá mudança do timestamp nos arquivos, logo o maven não terá porque re-compilar o código.

> 📋 Geralmente em pipelines queremos quebrar em estágios para dar mais visibilidade do que cada etapa faz, então essa solução, podemos dizer, não se aplica muito.

### Solução 2: tocar os .class / _touch_

No estágio que executa o empacotamento obtendo o source a parti do repo Git e o `target` a partir de estágio anterior, basta tocar todos os `.class`, assim a data dos `.class` será posterior a data dos `.java`, algo como:

```bash
find target/ -name "*.class" -exec touch -c {} \+
mvn package
```

### Solução 3: usar a propriedade `-DlastModGranularityMs` do plugin de compilação

Conforma a documentação do plugin `maven-compiler-plugin`:

> Sets the granularity in milliseconds of the last modification date for testing whether a source needs recompilation

Ou seja, nela informamos o máximo de tempo tolerado de defasgem entre a data de modificação do `.class` e do seu respectivo source (`.java`). Por mais que o `.java` esteja com data de modificação posterior a do `.class`, se estiver dentro do limite espeficiado na opção `lastModGranularityMs`, o plugin não irá re-compilar o `.java`.

```bash
mvn -DlastModGranularityMs=3600000 package
```

> 📋 `1h * 3600 * 1000 = 3600000 ms`

## Referências

- <https://maven.apache.org/plugins/maven-compiler-plugin/compile-mojo.html>
- <https://freedumbytes.bitbucket.io/dpl/html/continuous.integration.pipeline.html>
