---
layout: post
title:  "JBoss EAP 6 - Consumo elevado de CPU com muitos logs"
date: 2020-05-06 12:30:00 -03:00
tags:
  - jboss
author-id: mhagnumdw
image: "assets/img/posts/jboss-consumo-elevado-cpu-com-muitos-logs/banner.png"
feature-img: "assets/img/posts/jboss-consumo-elevado-cpu-com-muitos-logs/banner.png"
thumbnail: "assets/img/posts/jboss-consumo-elevado-cpu-com-muitos-logs/banner.png"
---

JBoss EAP 6 - Consumo elevado de CPU quando existem muitos arquivos arquivos de log, inclusive logs compactados.

<!--more-->

## Ambiente de teste

- RHEL 7
- JBoss EAP 6.4.20
- JDK 1.8.0_171

## Solução

Diminuir a quantidade de arquivos de logs. É isso mesmo, camarada! Esse problema talvez tenha sido corrigido na versão EAP 7.1.0+.

A pasta de logs pode ser descoberta a partir da system property `${jboss.server.log.dir}`.

## Como saber se o que causa 100% de CPU são mesmo os logs?

- Descobrir o PID do JBoss: `ps aux | grep -i java`
- Listar as threads desse PID que mais consomem CPU: `COLUMNS=10000 top -n 1 -b -H -p PID | head -30`
- Obter o _thread dump_ da JVM do JBoss: `jstack -l PID | vim -n -`
- Vincular o thread ID (TID) do `top` com o output do `jstack`: converter o TID do `top` de base 10 para base 16 e buscar por isso no `jstack`, assim achando o stack da respectiva thread
- Se houverem muitos stacks **parecidos entre si e parecidos com o apresentado abaixo**, provavelmente o excesso de logs causa o elevado consumo de CPU por essas threads

> Observar que o stack já é bem sugestivo com relação a _logging_

Stack trace

```raw
2020-05-05 18:28:44
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.171-b11 mixed mode):

"pool-3-thread-95" #373 prio=5 os_prio=0 tid=0x00007f3e0006d800 nid=0x1ba4 runnable [0x00007f3dcca9c000]
   java.lang.Thread.State: RUNNABLE
        at java.io.UnixFileSystem.checkAccess(Native Method)
        at java.io.File.canRead(File.java:768)
        at org.jboss.as.logging.LoggingResource$AllowedFilesFilter.accept(LoggingResource.java:274)
        at java.io.File.listFiles(File.java:1291)
        at org.jboss.as.logging.LoggingResource.findFiles(LoggingResource.java:217)
        at org.jboss.as.logging.LoggingResource.getChildrenNames(LoggingResource.java:147)
        at org.jboss.as.logging.LoggingResource.hasReadableFile(LoggingResource.java:203)
        at org.jboss.as.logging.LoggingResource.getChild(LoggingResource.java:98)
        at org.jboss.as.controller.OperationContextImpl.getAuthorizationResource(OperationContextImpl.java:1405)
        at org.jboss.as.controller.OperationContextImpl.getBasicAuthorizationResponse(OperationContextImpl.java:1362)
        at org.jboss.as.controller.OperationContextImpl.authorize(OperationContextImpl.java:1281)
        at org.jboss.as.controller.OperationContextImpl.readResourceFromRoot(OperationContextImpl.java:646)
        at org.jboss.as.controller.OperationContextImpl.readResourceFromRoot(OperationContextImpl.java:632)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler$ReadResourceDescriptionAccessControlContext.addParentResource(ReadResourceDescriptionHandler.java:786)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler$ReadResourceDescriptionAccessControlContext.getAllActualResourceAddresses(ReadResourceDescriptionHandler.java:774)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler$ReadResourceDescriptionAccessControlContext.getAllActualResourceAddresses(ReadResourceDescriptionHandler.java:778)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler$ReadResourceDescriptionAccessControlContext.getLocalResourceAddresses(ReadResourceDescriptionHandler.java:713)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler$ReadResourceDescriptionAccessControlContext.initLocalResourceAddresses(ReadResourceDescriptionHandler.java:702)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler$ReadResourceDescriptionAccessControlContext.access$200(ReadResourceDescriptionHandler.java:689)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler.doExecuteInternal(ReadResourceDescriptionHandler.java:204)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler.doExecute(ReadResourceDescriptionHandler.java:162)
        at org.jboss.as.controller.operations.global.ReadResourceDescriptionHandler.execute(ReadResourceDescriptionHandler.java:155)
        at org.jboss.as.controller.AbstractOperationContext.executeStep(AbstractOperationContext.java:710)
        at org.jboss.as.controller.AbstractOperationContext.doCompleteStep(AbstractOperationContext.java:545)
        at org.jboss.as.controller.AbstractOperationContext.completeStepInternal(AbstractOperationContext.java:338)
        at org.jboss.as.controller.AbstractOperationContext.executeOperation(AbstractOperationContext.java:314)
        at org.jboss.as.controller.OperationContextImpl.executeOperation(OperationContextImpl.java:1152)
        at org.jboss.as.controller.ModelControllerImpl.internalExecute(ModelControllerImpl.java:335)
        at org.jboss.as.controller.ModelControllerImpl.execute(ModelControllerImpl.java:191)
        at org.jboss.as.jmx.model.ResourceAccessControlUtil.getResourceAccess(ResourceAccessControlUtil.java:85)
        at org.jboss.as.jmx.model.RootResourceIterator.doIterate(RootResourceIterator.java:51)
        at org.jboss.as.jmx.model.RootResourceIterator.doIterate(RootResourceIterator.java:61)
        at org.jboss.as.jmx.model.RootResourceIterator.doIterate(RootResourceIterator.java:61)
        at org.jboss.as.jmx.model.RootResourceIterator.iterate(RootResourceIterator.java:43)
        at org.jboss.as.jmx.model.ModelControllerMBeanHelper.queryNames(ModelControllerMBeanHelper.java:175)
        at org.jboss.as.jmx.model.ModelControllerMBeanServerPlugin.queryNames(ModelControllerMBeanServerPlugin.java:209)
        at org.jboss.as.jmx.PluggableMBeanServerImpl.queryNames(PluggableMBeanServerImpl.java:806)
        at org.jboss.as.jmx.BlockingNotificationMBeanServer.queryNames(BlockingNotificationMBeanServer.java:133)
        at org.jboss.remotingjmx.protocol.v2.ServerProxy$QueryNamesHandler.handle(ServerProxy.java:1115)
        at org.jboss.remotingjmx.protocol.v2.ServerCommon$MessageReciever$1$1.run(ServerCommon.java:153)
        at org.jboss.as.jmx.ServerInterceptorFactory$Interceptor$1.run(ServerInterceptorFactory.java:75)
        at org.jboss.as.jmx.ServerInterceptorFactory$Interceptor$1.run(ServerInterceptorFactory.java:70)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:422)
        at org.jboss.as.controller.AccessAuditContext.doAs(AccessAuditContext.java:94)
        at org.jboss.as.jmx.ServerInterceptorFactory$Interceptor.handleEvent(ServerInterceptorFactory.java:70)
        at org.jboss.remotingjmx.protocol.v2.ServerCommon$MessageReciever$1.run(ServerCommon.java:149)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
```

> **NOTA:** o comando top exibe o ID da thread em base 10, enquanto o jstack exibe em base 16 (hexadecimal), por isso é necessária a conversão

> **NOTA:** no Windows a linha `at java.io.UnixFileSystem.checkAccess(Native Method)` deve ser `at java.io.WinNTFileSystem.checkAccess(Native Method)`

## Dicas

- Visualizar de forma resumida apenas as threads que possuem no seu stack um determinado pacote

```bash
jstack -l PID | grep -P --color '(^"|at br.com)'
```

## Referências

- <https://access.redhat.com/solutions/3031301>
