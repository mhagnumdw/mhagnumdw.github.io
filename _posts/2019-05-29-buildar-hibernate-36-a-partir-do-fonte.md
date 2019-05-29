---
layout: post
title:  "Buildar Hibernate 3.6.x"
date: 2019-05-29 08:30:00 -03:00
tags:
  - hibernate
author-id: mhagnumdw
feature-img: "assets/img/posts/buildar-hibernate-36-a-partir-do-fonte/buildar-hibernate-36-banner.png"
thumbnail: "assets/img/posts/buildar-hibernate-36-a-partir-do-fonte/buildar-hibernate-36-banner.png"
---

Buildar Hibernate 3.6.x a partir do fonte e versionar na sua infra.

<!--more-->

> Ambiente onde esse passo a passo foi realizado: Fedora 29
> 
> Opcional: testar esse passo a passo em um docker: `docker run -t -i fedora bash`

## Instalar dependências e clonar repositório
```shell
# instalar algumas dependências necessárias e úteis
dnf install git bash-completion iputils coreutils findutils which libnsl xmlstarlet wget vim

# clona o repositório oficial
git clone https://github.com/hibernate/hibernate-orm.git

# listar as branches remotas
git branch -r

# alternar para branch "3.6"
git checkout 3.6
```

## Verificar versão no pom.xml
Provavelmente a versão seja 3.6.11-SNAPSHOT

## Instalar JDK
Necessário jdk 5 e 6, mas com a 6 não funcionou, no lugar da jdk 6 usar a jdk 7
- baixar jdk 5: `jdk-1_5_0_22-linux-amd64.bin`
- baixar jdk 7: `jdk-7u80-linux-x64.rpm`

```shell
# instalar jdk 7
rpm -Uvh jdk-7u80-linux-x64.rpm
# verificar versão
java -version

# instalar jdk 5
./jdk-1_5_0_22-linux-amd64.bin
# verificar versão
~/jdk1.5.0_22/bin/java -version
```

## Gerar release
Poderia ser pelo plugin `maven-release-plugin` do maven, mas em razão da estrutura do projeto do hibernate essa abordagem não funciona. Vamos gerar então com um script shell disponibilizado dentro da pasta do próprio projeto.

```shell
# entrar na pasta do hibernate
cd hibernate-orm

# criar uma nova branch
git checkout -b 3.6.SEFIN 3.6

# configurar user e email dos commits
git config --local user.name "mhagnumdw" && git config --local user.email "mhagnumdw@gmail.com"

# gerar release: ao final o script pede pra fazer o push,
# mas como estamos fazendo no repositório do Hibernate do
# GitHub não temos acesso, bas então cancelar: ctrl + c.
# Se estiver usando um repositório seu, aceite o push.
# -r : versão da release
# -d : próxima versão de desenvolvimento
./tagRelease.sh -r 3.6.11.SEFINv2 -d 3.6.11.SEFINv3-SNAPSHOT

# ver a tag criada
git tag | grep -P '^3.6.'

# ver a diferença entre a branch 3.6 e a branch 3.6.SEFIN
# (essa última é a que estamos trabalhando)
git log --oneline origin/3.6..HEAD
```

## Baixar o Maven compatível com jdk 5
Tabela de compatibilidade: https://maven.apache.org/docs/history.html

```shell
# baixar
wget https://archive.apache.org/dist/maven/maven-3/3.1.1/binaries/apache-maven-3.1.1-bin.tar.gz
# descompactar
tar -xvzf apache-maven-3.1.1-bin.tar.gz
```

## Compilar / Empacotar / Deploy
Usar um `settings.xml` apropriado, que tenha os repositórios maven do JBoss e que tenha acesso ao Nexus caso seja feito deploy. Você pode usar a versão disponibilizada [aqui]({{ "/assets/img/posts/buildar-hibernate-36-a-partir-do-fonte/settings.xml" | relative_url }}) ou pegar a versão disponibilizada no item 4 na sessão referências.

### Abaixo três opções

#### Apenas empacotar (package)
_Não vai gerar o source pois o `maven-source-plugin` está definido para rodar no fase `verify`, ver `pom.xml`._
```shell
# executar
JAVA_HOME="/root/jdk1.5.0_22" \
  /root/apache-maven-3.1.1/bin/mvn -B -V \
  -gs /root/settings.xml \
  -DskipTests=true -Dmaven.test.skip=true \
  clean package \
  -DdisableDistribution=true \
  -Djdk16_home=/usr/java/jdk1.7.0_80
# verificar os jars gerados
find | grep -P -i '\.jar$'
```

#### Instalar (install)
```shell
# executar
JAVA_HOME="/root/jdk1.5.0_22" \
  /root/apache-maven-3.1.1/bin/mvn -B -V \
  -gs /root/settings.xml \
  -DskipTests=true -Dmaven.test.skip=true \
  clean install \
  -DdisableDistribution=true \
  -Djdk16_home=/usr/java/jdk1.7.0_80
# verificar os jars gerados
find /root/.m2/repository/org/hibernate/ | grep -P -i '\.jar$'
```

#### Deploy no Nexus
> Necessário mudar o `<distributionManagement>` no `./hibernate-parent/pom.xml`. O `settings.xml` tem que ter as credenciais para escrever no Nexus.

```shell
# executar
JAVA_HOME="/root/jdk1.5.0_22" \
  /root/apache-maven-3.1.1/bin/mvn -B -V \
  -gs /root/settings.xml \
  -DskipTests=true -Dmaven.test.skip=true \
  clean deploy \
  -DdisableDistribution=true \
  -Djdk16_home=/usr/java/jdk1.7.0_80
```

Bom... é isso.

## Referências
- [1] https://developer.jboss.org/thread/199389#jive-1168031045068142817703
- [2] https://hibernate.atlassian.net/browse/HHH-7682
- [3] https://github.com/hibernate/hibernate-orm/pull/393
- [4] https://developer.jboss.org/wiki/BuildingHibernateFromSource35
- [5] https://www.oracle.com/technetwork/java/archive-139210.html
- [6] https://bugzilla.redhat.com/show_bug.cgi?id=1653638


