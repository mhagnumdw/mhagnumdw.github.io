---
layout: post
title: 'Análise da memória da JVM em um container Docker'
date: 2021-01-02 12:18:00 -03:00
categories:
- jvm
tags:
- docker
- openshift
- kubernetes
- openjdk
author-id: mhagnumdw
image: "assets/img/posts/memoria-jvm-container-docker/banner.jpg"
feature-img: "assets/img/posts/memoria-jvm-container-docker/banner.jpg"
thumbnail: "assets/img/posts/memoria-jvm-container-docker/banner.jpg"
---

Análise da memória da JVM...

<!--more-->

- [Início](#início)
- [Relação entre o `Xmx` (`MaxHeapSize`) e a memória física total](#relação-entre-o-xmx-maxheapsize-e-a-memória-física-total)
  - [JVM antiga que identifica de forma errada a memória física total ⛔](#jvm-antiga-que-identifica-de-forma-errada-a-memória-física-total-)
  - [JVM que identifica de forma correta a memória física total 🎉](#jvm-que-identifica-de-forma-correta-a-memória-física-total-)
- [Alguns parâmetros da JVM menos comuns](#alguns-parâmetros-da-jvm-menos-comuns)
  - [`-XX:MaxRAM`](#-xxmaxram)
  - [`-XX:+UseParallelGC`](#-xxuseparallelgc)
  - [`-XX:MaxHeapFreeRatio=percent`](#-xxmaxheapfreeratiopercent)
  - [`-XX:GCTimeRatio=nnn`](#-xxgctimerationnn)
  - [`-XX:AdaptiveSizePolicyWeight=nn`](#-xxadaptivesizepolicyweightnn)
- [Forçando a jvm a devolver memória para o SO](#forçando-a-jvm-a-devolver-memória-para-o-so)
- [Várias JVM dentro de um container](#várias-jvm-dentro-de-um-container)
- [Referências](#referências)

## Início

**// TODO:** Tudo o que for dito aqui é valido para a OpenJDK, ok? Versão 1.8.0_212 acima, ok?

**// TODO:** renomear de HEAP para `HEAP`

However, as a starting point for running OpenJDK in a container, at least the following three memory-related tasks are key:

- Overriding the JVM maximum heap size.
- Encouraging the JVM to release unused memory to the operating system, if appropriate.
- Ensuring all JVM processes within a container are appropriately configured.

## Relação entre o `Xmx` (`MaxHeapSize`) e a memória física total

Por padrão, se o `Xmx` (`MaxHeapSize`) não for definido, o seu valor máximo será **1/4** da memória física total. Esse **1/4** é definido pelo parâmetro `MaxRAMFraction` e pode ser alterado.

**Para observar isso:**

- verificar a memória física total com o comando `free -m` (se a execução for dentro de um container docker, deve-se tomar como memória física total a memória alocada para o container)
- obter o `MaxRAMFraction` e `MaxHeapSize` da jvm, com o comando `java -XX:+PrintFlagsFinal -version | grep -iE 'HeapSize|PermSize|ThreadStackSize|MaxRAMFraction'`
- então verificar que se o `MaxRAMFraction` for **4**, o `MaxHeapSize` será **1/4** da memória física total

Abaixo uma ilustração do que foi dito acima:

![MaxRAMFraction vs MaxHeapSize vs Memória Física Total]({{ site.baseurl }}/assets/img/posts/memoria-jvm-container-docker/MaxRAMFraction-e-MaxHeapSize-vs-Memoria-Fisica-Total.png)

> 📋 ATENÇÃO
>
> - 👀 O comando `free -m` dentro de um container vai reportar a memória do host e não do container. O cálculo acima, no caso de um container, deve levar em consideração como memória física a memória alocada para o container.
> - 🐞 A jvm, em versões anteriores a **`// TODO: XXXXXXXXXXXXXXXXXX`**, só identifica a memória do host e não do container. Isso é um bug.
> - 😎 É possível verificar a memória física total de um container por meio do arquivo `cat /sys/fs/cgroup/memory/memory.limit_in_bytes` (para outros parâmetros ver [aqui](https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt))

### JVM antiga que identifica de forma errada a memória física total ⛔

Aqui vamos apenas demonstrar esse problema. Verificando a memória do host:

```console
$ free -m # valores em MB
              total        used        free      shared  buff/cache   available
Mem:          12549        3236         266         274        9046        8773
Swap:          4096           2        4094
```

Alocando **1GB** de memória física para o container docker da jvm:

```console
$ docker run -it --rm \
  --memory $(( 1 * 1024 * 1024 * 1024 )) \
  openjdk:8u92-alpine \
  java -XX:+PrintFlagsFinal -version | grep -iE 'HeapSize|PermSize|ThreadStackSize|MaxRAMFraction'

     intx CompilerThreadStackSize                   = 0                                   {pd product}
    uintx DefaultMaxRAMFraction                     = 4                                   {product}
    uintx ErgoHeapSizeLimit                         = 0                                   {product}
    uintx HeapSizePerGCThread                       = 87241520                            {product}
    uintx InitialHeapSize                          := 207618048                           {product}
    uintx LargePageHeapSizeThreshold                = 134217728                           {product}
    uintx MaxHeapSize                              := 3290431488                          {product}
    uintx MaxRAMFraction                            = 4                                   {product}
     intx ThreadStackSize                           = 1024                                {pd product}
     intx VMThreadStackSize                         = 1024                                {pd product}
openjdk version "1.8.0_92-internal"
OpenJDK Runtime Environment (build 1.8.0_92-internal-alpine-r1-b14)
OpenJDK 64-Bit Server VM (build 25.92-b14, mixed mode)
```

Pelo resultado acima, o valor do `MaxHeapSize` é `3138 MB` (`3290431488` bytes / 1024 / 1024).

**3138 MB é 1/4 da memória física do host e não da memória física do container.** Portanto a jvm identificou errado, isso é um bug que foi corrigido a partir da versão **`// TODO: XXXXXXXXXXXXXXXXXX`**.

> 📋 NOTA
>
> - Observar que para o teste acima usamos a jdk 1.8.0_92. Imagem docker `openjdk:8u92-alpine`.
> - Esse problema é facilmente contornado se o `Xmx` for explicitamente definido na linha de comando da jvm. O que eu pessoalmente acho uma boa prática.
> - Se por algum motivo não se deseja definir o `Xmx`, basta informar para a jvm o total de memória disponível por meio do parâmetro `-XX:MaxRAM`, ex: `-XX:MaxRAM=1G`.

### JVM que identifica de forma correta a memória física total 🎉

Agora, apenas para exemplificar o caso de sucesso. Verificando a memória do host:

```console
$ free -m # valores em MB
              total        used        free      shared  buff/cache   available
Mem:          12549        3236         266         274        9046        8773
Swap:          4096           2        4094
```

Alocando **1GB** de memória física para o container docker da jvm:

```console
$ docker run -it --rm \
  --memory $(( 1 * 1024 * 1024 * 1024 )) \
  openjdk:8-jdk-alpine \
  java -XX:+PrintFlagsFinal -version | grep -iE 'HeapSize|PermSize|ThreadStackSize|MaxRAMFraction'

     intx CompilerThreadStackSize                   = 0                                   {pd product}
    uintx DefaultMaxRAMFraction                     = 4                                   {product}
    uintx ErgoHeapSizeLimit                         = 0                                   {product}
    uintx HeapSizePerGCThread                       = 87241520                            {product}
    uintx InitialHeapSize                          := 16777216                            {product}
    uintx LargePageHeapSizeThreshold                = 134217728                           {product}
    uintx MaxHeapSize                              := 268435456                           {product}
    uintx MaxRAMFraction                            = 4                                   {product}
     intx ThreadStackSize                           = 1024                                {pd product}
     intx VMThreadStackSize                         = 1024                                {pd product}
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (IcedTea 3.12.0) (Alpine 8.212.04-r0)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)
```

Pelo resultado acima, o valor do `MaxHeapSize` é `256 MB` (`268435456` bytes / 1024 / 1024).

**256 MB é exatamente 1/4 da memória física do container.** Portanto a jvm identificou corretamente.

> 📋 NOTA
>
> - Observar que para o teste acima usamos a jdk 1.8.0_212. Imagem docker `openjdk:8-jdk-alpine`.
> - Pessoalmente eu acho uma boa prática definir o `Xmx` na linha de comando da jvm, ao invés de deixar a jvm calcular.

## Alguns parâmetros da JVM menos comuns

### `-XX:MaxRAM`

Indica para a jvm o total de memória física disponível. Exemplo para definir 1 GB: `-XX:MaxRAM=1G`.

### `-XX:+UseParallelGC`

_The parallel collector (also known as the throughput collector) performs minor collections in parallel, which can significantly reduce garbage collection overhead. It is intended for applications with medium-sized to large-sized data sets that are run on multiprocessor or multithreaded hardware. The parallel collector is selected by default on certain hardware and operating system configurations, or can be explicitly enabled with the option `-XX:+UseParallelGC`._

### `-XX:MaxHeapFreeRatio=percent`

_Sets the maximum allowed percentage of free heap space (0 to 100) after a GC event. If free heap space expands above this value, then the heap will be shrunk. By default, this value is set to 70%. The following example shows how to set the maximum free heap ratio to 75%: `-XX:MaxHeapFreeRatio=75`_

### `-XX:GCTimeRatio=nnn`

_A hint to the virtual machine that it's desirable that not more than 1 / (1 + nnn) of the application execution time be spent in the collector. For example `-XX:GCTimeRatio=19` sets a goal of 5% of the total time for GC and throughput goal of 95%. That is, the application should get 19 times as much time as the collector. By default the value is 99, meaning the application should get at least 99 times as much time as the collector. That is, the collector should run for not more than 1% of the total time. This was selected as a good choice for server applications. A value that is too high will cause the size of the heap to grow to its maximum._

### `-XX:AdaptiveSizePolicyWeight=nn`

**// TODO: escrever**

## Forçando a jvm a devolver memória para o SO

> 🤔💭 Mas por quê?!

Em servidores dedicados, hosts físicos ou máquinas virtuais, é comum e muitos locais recomendam, que em produção `Xms` (InitialHeapSize) e `Xmx` (MaxHeapSize) sejam iguais. Isso evita a jvm perder tempo alocando memória e também evita mais a frente precisar alocar e não ter disponível. No startup, com `Xms` e `Xmx` iguais, toda a memória HEAP será alocada pela jvm para que fique disponível para a aplicação.

Quando estamos em um ambiente compartilhado, com dezenas de containers rodando em um único host, orchestrados, por exemplo, por um `OpenShift` ou `Kubernetes`, queremos otimizar o uso de recursos ao máximo, pois a memória que um container está alocando e não usando, poderia estar disponível para outro container que realmente precise.

Outra ponto comum é que a memória HEAP da jvm em geral é bem menor que a memória física total do servidor, então dificilmente a jvm vai usar mais memória do que a disponível no servidor.

Outra questão é que a memória da jvm não se resume a memória HEAP, existem várias outras áreas de memória chamadas _off-heap_ e elas também consomem memória do sistema operacional, sendo exemplos dessas áreas: Metaspace, Thread Stack, Code Cache, Run-Time Constant Pool / Symbol, Native Method Stacks e Native Byte Buffers. **// TODO: rever o Metaspace**

Uma forma de forçar a jvm a devolver (liberar) a memória para o SO, é com os parâmetros abaixo:

```console
-XX:+UseParallelGC
-XX:MinHeapFreeRatio=5 -XX:MaxHeapFreeRatio=10 -XX:GCTimeRatio=4
-XX:AdaptiveSizePolicyWeight=90.
```

**// TODO: ver na prática como isso se comporta**

## Várias JVM dentro de um container

Não é comum, mas um container pode rodar mais de uma jvm full time (durante toda a vida do container), ou mesmo o processo principal da jvm pode disparar um processo temporário em outra jvm. Imagine que a aplicação executa processos `maven`, aqui já temos então outra jvm concorrendo com a jvm principal. Se você está fazendo um _troubleshooting_ com aplicativos java de diagnóstico, como por exemplo `jps`, `jmap`, `jcmd`, todos eles rodam em uma jvm e logo estarão concorrendo em recursos com a jvm principal.

O ideal é que esse outros processos java sejam iniciados com parâmetros de memória bem definidos. Fora explicitar os parâmetros da jvm em cada chamada, uma forma de tentar garantir isso globalmente dentro do container, é definindo valores padrões para processos java por meio da variável de ambiente `JAVA_TOOL_OPTIONS`. A OpenJDK e a Oracle JDK respeitam essa variável.

Uma forma de constatar isso é iniciando um processo da jvm, em uma única linha de comando:

```console
$ docker run -it --rm \
  --env JAVA_TOOL_OPTIONS="-Xms6m -Xmx20m" \
  openjdk:8-jdk-alpine \
  java -XX:+PrintFlagsFinal -version | grep -iE 'InitialHeapSize|MaxHeapSize'

    uintx InitialHeapSize                          := 6291456                             {product}
    uintx MaxHeapSize                              := 20971520                            {product}
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (IcedTea 3.12.0) (Alpine 8.212.04-r0)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)

$ echo "InitialHeapSize: $(( 6291456 / 1024 / 1024 )) MB  /  MaxHeapSize: $(( 20971520 / 1024 / 1024 )) MB"
InitialHeapSize: 6 MB  /  MaxHeapSize: 20 MB
```

Ou podemos iniciar um container com o `sh` e executar os comandos dentro do container para constatar:

```console
# Iniciando o container docker sem qualquer parâmetro
$ docker run -it --rm openjdk:8-jdk-alpine sh

# Aqui já estamos dentro do container, no shell

# Executando uma aplicação java
/ # java -XX:+PrintFlagsFinal -version | grep -iE 'InitialHeapSize|MaxHeapSize'
    uintx InitialHeapSize                          := 207618048                           {product}
    uintx MaxHeapSize                              := 3290431488                          {product}
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (IcedTea 3.12.0) (Alpine 8.212.04-r0)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)

# Apenas exibindo os valores de Xms e Xmx obtidos (ver acima que não definimos esses valores)
/ # echo "InitialHeapSize: $(( 207618048 / 1024 / 1024 )) MB  /  MaxHeapSize: $(( 3290431488 / 1024 / 1024 )) MB"
InitialHeapSize: 198 MB  /  MaxHeapSize: 3138 MB

# Agora vamos definir a env JAVA_TOOL_OPTIONS com os valores de Xms e Xmx
/ # export JAVA_TOOL_OPTIONS="-Xms6m -Xmx20m"

# Executando a aplicação java novamente
/ # java -XX:+PrintFlagsFinal -version | grep -iE 'InitialHeapSize|MaxHeapSize'
Picked up JAVA_TOOL_OPTIONS: -Xms6m -Xmx20m
    uintx InitialHeapSize                          := 6291456                             {product}
    uintx MaxHeapSize                              := 20971520                            {product}
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (IcedTea 3.12.0) (Alpine 8.212.04-r0)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)

# E podemos ver que mesmo que o Xms e Xmx não tenham sido definidos na linha de comando, os valores de
# Xms e Xms definidos na env JAVA_TOOL_OPTIONS foram respeitados
/ # echo "InitialHeapSize: $(( 6291456 / 1024 / 1024 )) MB  /  MaxHeapSize: $(( 20971520 / 1024 / 1024 )) MB"
InitialHeapSize: 6 MB  /  MaxHeapSize: 20 MB
```

Vale ressaltar, que se uma aplicação iniciar com valores definidos na linha de comando, esses serão os valores efetivamente usados, e não os valores definidos na variável de ambiente `JAVA_TOOL_OPTIONS`. Se `JAVA_TOOL_OPTIONS="-Xms6m -Xmx20m"` e a aplicação inicia explicitando `-Xmx128m`, o `Xmx` efetivo será `128m`.

> 📋 Eu recomendo definir a variável de ambiente `JAVA_TOOL_OPTIONS`.





**// TODO: falar?**
- que é possível obter os valores de limits e requests de memória dentro do POD? https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-resource-configure.html#nodes-cluster-resource-configure-request-limit_nodes-cluster-resource-configure**
- /sys/fs/cgroup/memory/memory.oom_control: https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-resource-configure.html#clipboard-8
- containerStatuses do POD pra ver o último motivo do restart: https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-resource-configure.html#clipboard-17

## Referências

- Diversos parâmetros da JVM: <https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html>
- `-XX:GCTimeRatio=nnn` e outros parâmetros do GC: <https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gc-ergonomics.html>
- JVM + Configuring cluster memory to meet container memory and risk requirements: <https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-resource-configure.html>
