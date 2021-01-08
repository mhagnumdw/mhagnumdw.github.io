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

Análise da memória da JVM... // TODO: escrever mais

<!--more-->

- [Início](#início)
- [Relação entre o `Xmx` (`MaxHeapSize`) e a memória física total](#relação-entre-o-xmx-maxheapsize-e-a-memória-física-total)
  - [JVM antiga que identifica de forma errada a memória física total ⛔](#jvm-antiga-que-identifica-de-forma-errada-a-memória-física-total-)
  - [JVM que identifica de forma correta a memória física total 🎉](#jvm-que-identifica-de-forma-correta-a-memória-física-total-)
- [Alguns parâmetros da JVM menos comuns](#alguns-parâmetros-da-jvm-menos-comuns)
  - [-XX:MaxRAM](#-xxmaxram)
  - [-XX:+UseParallelGC](#-xxuseparallelgc)
  - [-XX:MinHeapFreeRatio=percent](#-xxminheapfreeratiopercent)
  - [-XX:MaxHeapFreeRatio=percent](#-xxmaxheapfreeratiopercent)
  - [-XX:GCTimeRatio=nnn](#-xxgctimerationnn)
  - [-XX:AdaptiveSizePolicyWeight=nn](#-xxadaptivesizepolicyweightnn)
- [Forçando a jvm a devolver memória para o SO](#forçando-a-jvm-a-devolver-memória-para-o-so)
- [Várias JVM dentro de um container](#várias-jvm-dentro-de-um-container)
- [Ajustando os parâmetros da JVM](#ajustando-os-parâmetros-da-jvm)
  - [Sobre oa parâmetros da JVM utilizados nos testes](#sobre-oa-parâmetros-da-jvm-utilizados-nos-testes)
  - [Sobre o log do GC](#sobre-o-log-do-gc)
  - [Verificando o tempo do GC - Exemplo 1](#verificando-o-tempo-do-gc---exemplo-1)
  - [Verificando o tempo do GC - Exemplo 2](#verificando-o-tempo-do-gc---exemplo-2)
- [Executando os testes de carga](#executando-os-testes-de-carga)
  - [Interface gráfica (JMeter GUI)](#interface-gráfica-jmeter-gui)
  - [Linha de comando (JMeter)](#linha-de-comando-jmeter)
  - [Linha de comando (Jmeter com Taurus)](#linha-de-comando-jmeter-com-taurus)
- [// TODO: Falar dessas coisas?](#-todo-falar-dessas-coisas)
- [Referências](#referências)

## Início

**// TODO:** Tudo o que for dito aqui é valido para a OpenJDK, ok? Versão 1.8.0_212 acima, ok?

**// TODO:** renomear de HEAP para `HEAP`

**// TODO:** usar a sigla SO ou o termo Sistema Operacional?

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

// TODO: escrever sobre esses parâmetros em português da forma mais clara possível

### -XX:MaxRAM

Indica para a jvm o total de memória física disponível. Exemplo para definir 1 GB: `-XX:MaxRAM=1G`.

### -XX:+UseParallelGC

_The parallel collector (also known as the throughput collector) performs minor collections in parallel, which can significantly reduce garbage collection overhead. It is intended for applications with medium-sized to large-sized data sets that are run on multiprocessor or multithreaded hardware. The parallel collector is selected by default on certain hardware and operating system configurations, or can be explicitly enabled with the option `-XX:+UseParallelGC`._

### -XX:MinHeapFreeRatio=percent

// TODO: escrever

### -XX:MaxHeapFreeRatio=percent

_Sets the maximum allowed percentage of free heap space (0 to 100) after a GC event. If free heap space expands above this value, then the heap will be shrunk. By default, this value is set to 70%. The following example shows how to set the maximum free heap ratio to 75%: `-XX:MaxHeapFreeRatio=75`_

### -XX:GCTimeRatio=nnn

_A hint to the virtual machine that it's desirable that not more than 1 / (1 + nnn) of the application execution time be spent in the collector. For example `-XX:GCTimeRatio=19` sets a goal of 5% of the total time for GC and throughput goal of 95%. That is, the application should get 19 times as much time as the collector. By default the value is 99, meaning the application should get at least 99 times as much time as the collector. That is, the collector should run for not more than 1% of the total time. This was selected as a good choice for server applications. A value that is too high will cause the size of the heap to grow to its maximum._

### -XX:AdaptiveSizePolicyWeight=nn

// TODO: escrever

## Forçando a jvm a devolver memória para o SO

> 🤔💭 Mas por quê?!

Em servidores dedicados, hosts físicos ou máquinas virtuais, é comum e muitos locais recomendam, que em produção `Xms` (InitialHeapSize) e `Xmx` (MaxHeapSize) sejam iguais. Isso evita a jvm perder tempo alocando memória e também evita mais a frente precisar alocar e não ter disponível. No startup, com `Xms` e `Xmx` iguais, toda a memória HEAP será alocada pela jvm para que fique disponível para a aplicação.

Quando estamos em um ambiente compartilhado, com dezenas de containers rodando em um único host, orchestrados, por exemplo, por um `OpenShift` ou `Kubernetes`, queremos otimizar o uso de recursos ao máximo, pois a memória que um container está alocando e não usando, poderia estar disponível para outro container que realmente precise.

Outra ponto comum é que a memória HEAP da jvm em geral é bem menor que a memória física total do servidor, então dificilmente a jvm vai usar mais memória do que a disponível no servidor.

Outra questão é que a memória da jvm não se resume a memória HEAP, existem várias outras áreas de memória chamadas _off-heap_ e elas também consomem memória do sistema operacional, sendo exemplos dessas áreas: Metaspace, Thread Stack, Code Cache, Run-Time Constant Pool / Symbol, Native Method Stacks e Native Byte Buffers.

// TODO: rever o Metaspace citado logo acima, se algumas das áreas de memórias já não são contidas nele

Uma forma de forçar a jvm a devolver (liberar) a memória para o SO, é com os parâmetros abaixo:

```console
-XX:+UseParallelGC
-XX:MinHeapFreeRatio=5 -XX:MaxHeapFreeRatio=10 -XX:GCTimeRatio=4
-XX:AdaptiveSizePolicyWeight=90.
```

// TODO: explicar um pouco aqui ou informar que isso será visto em detalhes mais a frente

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

// TODO: dividir a parte acima em blocos talvez melhore na visualização

Vale ressaltar, que se uma aplicação iniciar com valores definidos na linha de comando, esses serão os valores efetivamente usados, e não os valores definidos na variável de ambiente `JAVA_TOOL_OPTIONS`. Se `JAVA_TOOL_OPTIONS="-Xms6m -Xmx20m"` e a aplicação inicia explicitando `-Xmx128m`, o `Xmx` efetivo será `128m`.

> 📋 Eu recomendo definir a variável de ambiente `JAVA_TOOL_OPTIONS`.

## Ajustando os parâmetros da JVM

Mas não vamos fazer às cegas. Vamos fazer testes práticos e mostrar com números se os ajustes valem a pena ou não.

GC paralelo, na configuração padrão, a jvm vai tentar usar todo o heap. Isso acontece mesmo se a aplicação precisar de pouco espaço para executar. No GC serial isso é minimizado, mas ainda assim ocorre.

Se a aplicação precisa de bem menos memória do que o HEAP máximo configurado, o GC pode rodar antecipadamente, antes que o HEAP esteja quase todo em uso, liberando memória para o SO. Isso pode ser configurado!

Muitas vezes a única instrução que passamos para o GC é o máximo de HEAP que pode ser usado, não informando, por exemplo, que menos HEAP (memória) seja usado, e muito menos informamos como ele deve fazer isso.

Abaixo é um exemplo de uma aplicação em produção que está a 13 dias no ar, consumindo aproximadamente 380 MB da HEAP e após um Full GC forçado cai para 122 MB, liberando 258 MB. A memória só não caiu mais porque a aplicação estava com algumas tarefas em andamento no momento do GC. Em uma execução posterior forçada do GC a memória caiu para 81 MB de uso.

> ⚠️ Um Full GC pode parar a sua apicação por milésemos ou segundos! Cuidado!

<video muted autoplay controls style="width=:100%;padding: unset;">
    <source src="{{ site.baseurl }}/assets/img/posts/memoria-jvm-container-docker/heap-after-force-gc.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

Como dito acima, é possível configurar a jvm para que menos HEAP seja usado, se possível for, claro, liberando memória para o SO, deixando o HEAP de um tamanho próximo aos dados que a aplicação efetivamente precisa no momento (ou seja, deixando em memória apenas objetos ainda referenciados pela aplicação/jvm).

Precisamos começar definindo o `-Xms`, algo como `-Xms48M`. Isso varia com a aplicação. Se a aplicação inicialmente precisa de um mínimo de **100 MB**, faz sentido definir algo como `-Xms128M`. Para uma necessidade inicial de **100 MB** pode-se definir `-Xms48M`, mas o GC só vai perder tempo alocando mais HEAP.

Outra opção que temos é aumentar a frequência do GC, para que objetos mortos (não mais referenciados) possam ser coletados e memória possa ser liberada, mas de modo que isso adicione um custo baixo no processamento do GC para que não impacte no funcionamento da aplicação. Vale lembrar que as threads de limpeza do GC executam em paralelo com as threads da aplicação.

Em servidores JBoss que tenho acesso, que recebem muitas requisições por dia, o tempo do GC fica abaixo de **1%** do tempo total de execução da JVM. Então se pudermos aumentar a frequência do GC de modo que não aumente muito esse tempo, estaremos removendo com mais frequência objetos mortos e consequentemente menos memória é alocada (usada).

A JVM (Oracle JDK, OpenJDK, outras?) por padrão não é configurada para ser mais agressiva no GC porque inicialmente as aplicações rodavam em servidores dedicados - inclusive é o caso de algumas aplicações enterprise que conheço, com todos os recursos voltados para a aplicação, onde qualquer ganho de performance, embora ínfimo, era (ou é!) levado em consideração. Certamente alguns aplicativos, dependendo de como eles precisem usar a memória, não irão se beneficiar de um GC mais agressivo, pelo contrário, pode ser que a performance degrade muito devido a concorrência entre o GC (tentando limpar a memória) e a aplicação (tentando fazer o seu trabalho).

### Sobre oa parâmetros da JVM utilizados nos testes

Para os testes vamos usar 4 configurações de GC:

1. -XX:UseParallelGC
2. -XX:+UseSerialGC
3. -XX:+UseSerialGC -XX:MinHeapFreeRatio=20 -XX:MaxHeapFreeRatio=40
4. -XX:UseParallelGC -XX:MinHeapFreeRatio=20 -XX:MaxHeapFreeRatio=40 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90

`-XX:UseParallelGC` utiliza várias threads para fazer a coleta de lixo, enquanto `-XX:+UseSerialGC` utiliza apenas uma thread.

O GC pode decidir por alocar mais HEAP ou mesmo liberar HEAP e isso pode ser controlado por meio dos parâmetros `-XX:MinHeapFreeRatio` e `-XX:MaxHeapFreeRatio`, que recebem um valor inteiro em porcentagem.

- `-XX:MinHeapFreeRatio`: porcentagem mínima de espaço livre na HEAP (0 a 100%) **após uma coleta do GC**. Se o tamanho do HEAP ficar menor que esse valor, o HEAP será expandido. O valor padrão é 40%. Exemplo: para um valor de 20% e HEAP com tamanho máximo de 256 MB, o HEAP mínimo alocado deve ser 51,2 MB. Se após o GC o HEAP ficar em 40 MB, ele será expandido para um valor acima de 51,2 MB.
- // TODO: acho que tanto o MinHeapFreeRatio (acima) e MaxHeapFreeRatio (abaixo), são em relação ao espaço livre da HEAP e não em relação ao tamanho máximo. Será? Ver isso! Mas agora ao ler isso <https://www.openshift.com/blog/scaling-java-containers> mudei mais uma vez de ideia.
- `-XX:MaxHeapFreeRatio`: porcentagem máxima de espaço livre na HEAP (0 a 100%) **após uma coleta do GC**. Se o tamanho do HEAP ficar maior que esse valor, o HEAP será reduzido. O valor padrão é 70%. Exemplo: para um valor de 40% e HEAP com tamanho máximo de 256 MB, o HEAP máximo alocado deve ser 102,4 MB.

O GC paralelo (`-XX:UseParallelGC`) tenta equilibrar o custo de fazer a limpeza da memória com o tempo gasto. Supondo uma mesma taxa de alocação de objetos, em um HEAP grande o GC acaba agindo menos, em um HEAP pequeno, como a memória enche mais rápido, o GC precisa agir mais. É possível balancear esses dois pontos, custo do GC vs tempo para fazer a limpeza, por meio dos parâmetros `-XX:GCTimeRatio` (padrão: 99) e `-XX:AdaptiveSizePolicyWeight` (padrão: 10 [0-100%]).

```bash
# visualizar o valor padrão
java -XX:+PrintFlagsFinal -version | grep -iE 'AdaptiveSizePolicyWeight|GCTimeRatio'

# ou para ver o valor de uma jvm já em execução
jcmd <JVM_PID> VM.flags -all | grep -iE 'AdaptiveSizePolicyWeight|GCTimeRatio'
```

- `-XX:GCTimeRatio`: é uma dica para a jvm de que é desejável que não mais que `1 / (1 + nnn)` do tempo de execução da jvm seja gasto com o GC. O valor padrão, 99, implica que o GC só pode tomar no máximo **1%** (`1 / 1 + 99`) do tempo total de execução da jvm. Setando `-XX:GCTimeRatio` para, por exemplo, 4, teremos até 20% do tempo para o GC, ou seja, dedica mais tempo para limpeza do que para a redução de tempo de limpeza. Vale lembrar que esses 20% do tempo para o GC é o pior caso, não quer dizer que ele sempre usará 20% do tempo para fazer limpeza, isso até tornaria impraticável o uso da aplicação que "só" teria 80% do tempo.
- `-XX:AdaptiveSizePolicyWeight`: indica quanto tempo das execuções anteriores do GC devem ser levadas em consideração para atingir a meta de tempo. O valor padrão, 10, indica que 90% das execuções anteriores devem ser levadas em consideração e apenas 10% deve levar em consideração a execução atual. Supondo um valor 90 - ao invés do padrão 10 - indica que a meta de tempo será mais baseada na execução corrente do GC do que nas execuções anteriores. Supõe-se que o GC corrente leva menos tempo que a soma dos anteriores, logo o GC fica mais longe da meta de tempo definida pelo `-XX:GCTimeRatio`, logo o GC acaba agindo mais.

### Sobre o log do GC

Para decidirmos qual das 4 opções é a melhor, ou melhor falando, é a que mais se encaixa para a carga de trabalho a qual a aplicação será submetida, vamos analisar o log do GC. É possível atachar na JVM um programa (um jar) que se conecta direto ao GC por meio do [JVM Tool Interface/JVMTI](https://docs.oracle.com/javase/8/docs/platform/jvmti/jvmti.html), coletando dados direto do GC e fazendo análise que for. Não vamos por essa opção. Achei mais simples ativar o log de texto do GC, por meio do parâmetro `-Xloggc:/path/to/gc.log` e um script `shell` simples que eu criei fazer a análise. A desvantagem do log de texto é que não há um padrão para esse log entre as implementações de jvm. Então pode ser que esse script não funcione em outra versão/vendor de jvm.

// TODO: acima eu digo que usei o log do gc, mas só usei em partes, usei em conjunto o log do jstat -gc "$JVM_PID"


### Verificando o tempo do GC - Exemplo 1

// TODO: colocar aqui os tempos de GC do GRPFOR ou do ISS Fortaleza após ficarem no ar por um longo tempo. Isso é pra mostrar a razão entre o tempo de GC e o tempo total da aplicação.

### Verificando o tempo do GC - Exemplo 2

Outro exemplo, de um JBoss EAP 6.4, rodando a **108 dias**, mas com quase nada de carga nesse período, apenas com os seguintes parâmetros configurados referentes a memória e GC: `-XX:PermSize=256m -XX:MaxPermSize=256m -Xms1024m -Xmx2048m`. Com um tempo de GC de **6974 segundos (~ 2h)**, o que representa apenas **0,0746%** do tempo total de execução da jvm. Abaixo como os dados desse exemplo foram coletados:

Obter o PID da JVM:

```bash
jps -lvm
# ou
ps aux | grep java
```

Vamos supor o PID 32549.

Obter o uptime da JVM em segundos:

```console
$ jcmd 32549 VM.uptime
32549:
9347114.094 s
```

Obter dados do GC da JVM (a coluna GCT significa _Garbage Collection Time_, em segundos):

```console
$ jstat -gc 32549
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
25088,0 25088,0 23648,1  0,0   648704,0 182309,7 1398272,0  1255760,7  251504,0 231526,4 29608,0 24808,2  24832 6971,431   6      3,433 6974,864
```

Temos **6974 segundos**.

Descobrir quantos % do tempo o GC levou do tempo total de execução da JVM. Uma matemática simples:

```console
$ bc <<< "scale=8; 6974 / 9347114 * 100"
.07461100
```

Temos **0.07461100%**. É muito pouco!

> 🧙‍♂️ Em um comando só:
>
> ```bash
> JVM_PID=32549; \
> GC_TOTAL_TIME=$( jstat -gc $JVM_PID | tail -n 1 | awk '{print $17}' | grep -P -o '^\d+' ); \
> JVM_UPTIME=$( jcmd $JVM_PID VM.uptime | tail -1 | grep -P -o '^\d+' ); \
> bc <<< "scale=8; $GC_TOTAL_TIME / $JVM_UPTIME * 100"
> ```

## Executando os testes de carga

Os testes de carga são escritos no [JMeter](https://jmeter.apache.org/) e estão definidos no arquivo `jmeter.jmx`.

### Interface gráfica (JMeter GUI)

Os testes pode ser executados pela interface gráfica (GUI) do JMeter, **mas nesse caso apenas em tempo de desenvolvimento dos testes, pois a interface gráfica acaba consumindo recursos e concorrendo com o teste em si**. Para executar basta abrir o JMeter, menu `File > Open`, selecionar o arquivo `jmeter.jmx`, checar se os parâmetros nos itens globais `User Defined Variables`, `HTTP Request Defaults` e `Grupo de Usuários (Thread Group)` estão corretos, e por fim clicar no botão `Start` (uma seta verde no formato de Play ▶).

### Linha de comando (JMeter)

Os testes em sua execução valendo devem ser feitos via linha de comando, em uma máquina que esteja executando apenas os testes, sem qualquer outro processamento em paralelo que possa degradar a performance do teste. Para iniciar o teste basta executar:

```bash
jmeter -n -t jmeter.jmx
```

Se desejar ver um pouco mais de detalhes das requisições e da execução do JMeter, basta ativar o log de ambas, assim:

```bash
jmeter -n -t jmeter.jmx \
  -l /tmp/jmeter-requests.jtl -j /tmp/jmeter.log
```

Mais parâmetros de linha de comando: <https://jmeter.apache.org/usermanual/get-started.html#non_gui>

### Linha de comando (Jmeter com [Taurus](https://gettaurus.org/))

Aqui temos uma interface bonita no console, que permite acompanhar os testes em tempo real.

```bash
docker run -it --rm \
  -v ~/projetos/jarvis:/bzt-configs \
  -v /tmp/jarvis-load-test-artifacts:/tmp/artifacts \
  blazemeter/taurus jmeter.jmx
```

> 📋 NOTA
>
> - `~/projetos/jarvis` é onde se encontra o `jmeter.jmx`
> - o volume `/tmp/jarvis-load-test-artifacts` é opcional
> - em sistemas com SELinux é preciso aplicar o contexto nas pastas dos volumes, ex: `sudo chcon -t svirt_sandbox_file_t /tmp/jarvis-load-test-artifacts`

## // TODO: Falar dessas coisas?

- que é possível obter os valores de limits e requests de memória dentro do POD? <https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-resource-configure.html#nodes-cluster-resource-configure-request-limit_nodes-cluster-resource-configure>**
- /sys/fs/cgroup/memory/memory.oom_control: <https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-resource-configure.html#clipboard-8>
- containerStatuses do POD pra ver o último motivo do restart: <https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-resource-configure.html#clipboard-17>

## Referências

- Diversos parâmetros da JVM: <https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html>
- `-XX:GCTimeRatio=nnn` e outros parâmetros do GC: <https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gc-ergonomics.html>
- JVM + Configuring cluster memory to meet container memory and risk requirements: <https://docs.openshift.com/container-platform/4.6/nodes/clusters/nodes-cluster-resource-configure.html>
- Tuning Java's footprint in OpenShift (Part 1): <https://developers.redhat.com/blog/2014/07/15/dude-wheres-my-paas-memory-tuning-javas-footprint-in-openshift-part-1/>
- Tuning Java’s footprint in OpenShift (Part 2): <https://developers.redhat.com/blog/2014/07/22/dude-wheres-my-paas-memory-tuning-javas-footprint-in-openshift-part-2/>
- <https://www.baeldung.com/jvm-garbage-collectors>
- <https://www.baeldung.com/jvm-parameters>
- <https://www.blazemeter.com/blog/3-easy-ways-to-monitor-jmeter-non-gui-test-results>
