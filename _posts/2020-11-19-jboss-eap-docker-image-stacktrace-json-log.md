---
layout: post
title: 'Imagem docker JBoss EAP 6.4: Logs em JSON e stacktrace'
date: 2020-11-19 19:20:00 -03:00
categories:
- JBoss EAP
tags:
- docker
- jboss
author-id: mhagnumdw
image: "assets/img/posts/jboss-eap-docker-image-stacktrace-json-log/banner.png"
feature-img: "assets/img/posts/jboss-eap-docker-image-stacktrace-json-log/banner.png"
thumbnail: "assets/img/posts/jboss-eap-docker-image-stacktrace-json-log/banner.png"
---

Ativar o log no formato JSON na imagem docker do JBoss EAP 6.4. Também vamos mudar o formato que o stacktrace aparece no JSON.

<!--more-->

## Imagem oficial

As imagens docker oficiais podem ser obtidas no registry da RedHat, em `registry.redhat.io`.

```bash
# login
docker login -u USER registry.redhat.io

# listar as tags disponíveis
skopeo list-tags docker://registry.redhat.io/jboss-eap-6/eap64-openshift
```

## Ativando o log JSON

Para ativar o log do JBoss no formato JSON, basta subir um container setando a variável de ambiente `ENABLE_JSON_LOGGING` para `true`.

```bash
# executar
docker run --rm \
  --env ENABLE_JSON_LOGGING=true \
  registry.redhat.io/jboss-eap-6/eap64-openshift:1.9

# ou se você quiser ver o JSON formatado
jq -R '. as $raw | try fromjson catch $raw' <(\
  docker run --rm \
    --env ENABLE_JSON_LOGGING=true \
    registry.redhat.io/jboss-eap-6/eap64-openshift:1.9 \
)
```

A partir disso cada linha do log da aplicação será parecida com isso:

```json
{
  "@version": 1,
  "@timestamp": "2020-11-20T14:20:41-03:00",
  "sequence": 7229,
  "loggerClassName": "org.jboss.as.server.ServerLogger_$logger",
  "loggerName": "org.jboss.as",
  "level": "INFO",
  "message": "JBAS015874: JBoss EAP 6.4.20.GA (AS 7.5.20.Final-redhat-1) iniciado em 65151ms - Iniciado 16089 de serviços 16123 (os serviços 121 são lazy, passivos ou em demanda)",
  "threadName": "Controller Boot Thread",
  "threadId": 19,
  "mdc": {},
  "ndc": "",
  "log-handler": "CONSOLE"
}
```

**Se houver exceção, o log será parecido com isso:**

<details>
  <summary>Clique AQUI para visualizar o log em JSON</summary>
<!-- Não mudar o bloco de código para ```json , pois a formatação
dentro do details+summary só funcionou com o bloco de código do Jekyll -->
{% highlight json %}
{
  "@version": 1,
  "@timestamp": "2020-11-20T16:24:35-03:00",
  "sequence": 4998,
  "loggerClassName": "org.jboss.jca.core.CoreLogger_$logger",
  "loggerName": "org.jboss.jca.core.connectionmanager.pool.strategy.OnePool",
  "level": "WARN",
  "message": "IJ000604: Throwable while attempting to get a new connection: null",
  "threadName": "ServerService Thread Pool -- 62",
  "threadId": 126,
  "mdc": {},
  "ndc": "",
  "exception": {
    "refId": 1,
    "exceptionType": "javax.resource.ResourceException",
    "message": "Could not create connection",
    "frames": [
      {
        "class": "org.jboss.jca.adapters.jdbc.local.LocalManagedConnectionFactory",
        "method": "getLocalManagedConnection",
        "line": 351
      },
      {
        "class": "org.jboss.jca.adapters.jdbc.local.LocalManagedConnectionFactory",
        "method": "createManagedConnection",
        "line": 299
      },
      {
        "class": "org.jboss.jca.core.connectionmanager.pool.mcp.SemaphoreArrayListManagedConnectionPool",
        "method": "createConnectionEventListener",
        "line": 874
      },
      {
        "class": "org.jboss.jca.core.connectionmanager.pool.mcp.SemaphoreArrayListManagedConnectionPool",
        "method": "getConnection",
        "line": 416
      },
      {
        "class": "org.jboss.jca.core.connectionmanager.pool.AbstractPool",
        "method": "getSimpleConnection",
        "line": 479
      },
      {
        "class": "org.jboss.jca.core.connectionmanager.pool.AbstractPool",
        "method": "getConnection",
        "line": 451
      },
      {
        "class": "org.jboss.jca.core.connectionmanager.AbstractConnectionManager",
        "method": "getManagedConnection",
        "line": 344
      },
      {
        "class": "org.jboss.jca.core.connectionmanager.tx.TxConnectionManagerImpl",
        "method": "getManagedConnection",
        "line": 367
      },
      {
        "class": "org.jboss.jca.core.connectionmanager.AbstractConnectionManager",
        "method": "allocateConnection",
        "line": 499
      },
      {
        "class": "org.jboss.jca.adapters.jdbc.WrapperDataSource",
        "method": "getConnection",
        "line": 143
      },
      {
        "class": "org.jboss.as.connector.subsystems.datasources.WildFlyDataSource",
        "method": "getConnection",
        "line": 69
      },
      {
        "class": "org.hibernate.ejb.connection.InjectedDataSourceConnectionProvider",
        "method": "getConnection",
        "line": 71
      },
      {
        "class": "org.hibernate.cfg.SettingsFactory",
        "method": "buildSettings",
        "line": 113
      },
      {
        "class": "org.hibernate.cfg.Configuration",
        "method": "buildSettingsInternal",
        "line": 2863
      },
      {
        "class": "org.hibernate.cfg.Configuration",
        "method": "buildSettings",
        "line": 2859
      },
      {
        "class": "org.hibernate.cfg.Configuration",
        "method": "buildSessionFactory",
        "line": 1870
      },
      {
        "class": "org.hibernate.ejb.Ejb3Configuration",
        "method": "buildEntityManagerFactory",
        "line": 906
      },
      {
        "class": "org.hibernate.ejb.HibernatePersistence",
        "method": "createContainerEntityManagerFactory",
        "line": 74
      },
      {
        "class": "org.jboss.as.jpa.service.PersistenceUnitServiceImpl",
        "method": "createContainerEntityManagerFactory",
        "line": 226
      },
      {
        "class": "org.jboss.as.jpa.service.PersistenceUnitServiceImpl",
        "method": "access$700",
        "line": 59
      },
      {
        "class": "org.jboss.as.jpa.service.PersistenceUnitServiceImpl$1",
        "method": "run",
        "line": 107
      },
      {
        "class": "java.util.concurrent.ThreadPoolExecutor",
        "method": "runWorker",
        "line": 1149
      },
      {
        "class": "java.util.concurrent.ThreadPoolExecutor$Worker",
        "method": "run",
        "line": 624
      },
      {
        "class": "java.lang.Thread",
        "method": "run",
        "line": 748
      },
      {
        "class": "org.jboss.threads.JBossThread",
        "method": "run",
        "line": 122
      }
    ],
    "causedBy": {
      "exception": {
        "refId": 2,
        "exceptionType": "java.sql.SQLException",
        "message": "ORA-01017: invalid username/password; logon denied\n",
        "frames": [
          {
            "class": "oracle.jdbc.driver.T4CTTIoer11",
            "method": "processError",
            "line": 494
          },
          {
            "class": "oracle.jdbc.driver.T4CTTIoer11",
            "method": "processError",
            "line": 441
          },
          {
            "class": "oracle.jdbc.driver.T4CTTIoer11",
            "method": "processError",
            "line": 436
          },
          {
            "class": "oracle.jdbc.driver.T4CTTIfun",
            "method": "processError",
            "line": 1061
          },
          {
            "class": "oracle.jdbc.driver.T4CTTIoauthenticate",
            "method": "processError",
            "line": 550
          },
          {
            "class": "oracle.jdbc.driver.T4CTTIfun",
            "method": "receive",
            "line": 623
          },
          {
            "class": "oracle.jdbc.driver.T4CTTIfun",
            "method": "doRPC",
            "line": 252
          },
          {
            "class": "oracle.jdbc.driver.T4CTTIoauthenticate",
            "method": "doOAUTH",
            "line": 499
          },
          {
            "class": "oracle.jdbc.driver.T4CTTIoauthenticate",
            "method": "doOAUTH",
            "line": 1279
          },
          {
            "class": "oracle.jdbc.driver.T4CConnection",
            "method": "logon",
            "line": 663
          },
          {
            "class": "oracle.jdbc.driver.PhysicalConnection",
            "method": "connect",
            "line": 688
          },
          {
            "class": "oracle.jdbc.driver.T4CDriverExtension",
            "method": "getConnection",
            "line": 39
          },
          {
            "class": "oracle.jdbc.driver.OracleDriver",
            "method": "connect",
            "line": 691
          },
          {
            "class": "org.jboss.jca.adapters.jdbc.local.LocalManagedConnectionFactory",
            "method": "getLocalManagedConnection",
            "line": 323
          },
          {
            "class": "org.jboss.jca.adapters.jdbc.local.LocalManagedConnectionFactory",
            "method": "createManagedConnection",
            "line": 299
          },
          {
            "class": "org.jboss.jca.core.connectionmanager.pool.mcp.SemaphoreArrayListManagedConnectionPool",
            "method": "createConnectionEventListener",
            "line": 874
          },
          {
            "class": "org.jboss.jca.core.connectionmanager.pool.mcp.SemaphoreArrayListManagedConnectionPool",
            "method": "getConnection",
            "line": 416
          },
          {
            "class": "org.jboss.jca.core.connectionmanager.pool.AbstractPool",
            "method": "getSimpleConnection",
            "line": 479
          },
          {
            "class": "org.jboss.jca.core.connectionmanager.pool.AbstractPool",
            "method": "getConnection",
            "line": 451
          },
          {
            "class": "org.jboss.jca.core.connectionmanager.AbstractConnectionManager",
            "method": "getManagedConnection",
            "line": 344
          },
          {
            "class": "org.jboss.jca.core.connectionmanager.tx.TxConnectionManagerImpl",
            "method": "getManagedConnection",
            "line": 367
          },
          {
            "class": "org.jboss.jca.core.connectionmanager.AbstractConnectionManager",
            "method": "allocateConnection",
            "line": 499
          },
          {
            "class": "org.jboss.jca.adapters.jdbc.WrapperDataSource",
            "method": "getConnection",
            "line": 143
          },
          {
            "class": "org.jboss.as.connector.subsystems.datasources.WildFlyDataSource",
            "method": "getConnection",
            "line": 69
          },
          {
            "class": "org.hibernate.ejb.connection.InjectedDataSourceConnectionProvider",
            "method": "getConnection",
            "line": 71
          },
          {
            "class": "org.hibernate.cfg.SettingsFactory",
            "method": "buildSettings",
            "line": 113
          },
          {
            "class": "org.hibernate.cfg.Configuration",
            "method": "buildSettingsInternal",
            "line": 2863
          },
          {
            "class": "org.hibernate.cfg.Configuration",
            "method": "buildSettings",
            "line": 2859
          },
          {
            "class": "org.hibernate.cfg.Configuration",
            "method": "buildSessionFactory",
            "line": 1870
          },
          {
            "class": "org.hibernate.ejb.Ejb3Configuration",
            "method": "buildEntityManagerFactory",
            "line": 906
          },
          {
            "class": "org.hibernate.ejb.HibernatePersistence",
            "method": "createContainerEntityManagerFactory",
            "line": 74
          },
          {
            "class": "org.jboss.as.jpa.service.PersistenceUnitServiceImpl",
            "method": "createContainerEntityManagerFactory",
            "line": 226
          },
          {
            "class": "org.jboss.as.jpa.service.PersistenceUnitServiceImpl",
            "method": "access$700",
            "line": 59
          },
          {
            "class": "org.jboss.as.jpa.service.PersistenceUnitServiceImpl$1",
            "method": "run",
            "line": 107
          },
          {
            "class": "java.util.concurrent.ThreadPoolExecutor",
            "method": "runWorker",
            "line": 1149
          },
          {
            "class": "java.util.concurrent.ThreadPoolExecutor$Worker",
            "method": "run",
            "line": 624
          },
          {
            "class": "java.lang.Thread",
            "method": "run",
            "line": 748
          },
          {
            "class": "org.jboss.threads.JBossThread",
            "method": "run",
            "line": 122
          }
        ]
      }
    }
  },
  "log-handler": "CONSOLE"
}
{% endhighlight %}
</details>

> 📋 Observar que cada linha do stacktrace ficou em um fragmento de JSON, o que pode deixar um pouco chato a visualização.

Podemos configurar para que o stacktrace da exception fique no formato padrão, de quando visualizamos no console.

## Formato do stacktrace da exception no log

Vamos mudar para o formato tradicional, algo parecido com:

```text
org.jboss.msc.service.StartException in service jboss.web.deployment.default-host./xxx: org.jboss.msc.service.StartException in anonymous service: JBAS018040: Falha ao iniciar o contexto
    at org.jboss.as.web.deployment.WebDeploymentService$1.run(WebDeploymentService.java:99)
    at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
    at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
    at java.lang.Thread.run(Thread.java:748)
    at org.jboss.threads.JBossThread.run(JBossThread.java:122)
Caused by: org.jboss.msc.service.StartException in anonymous service: JBAS018040: Falha ao iniciar o contexto
    at org.jboss.as.web.deployment.WebDeploymentService.doStart(WebDeploymentService.java:168)
    at org.jboss.as.web.deployment.WebDeploymentService.access$000(WebDeploymentService.java:61)
    at org.jboss.as.web.deployment.WebDeploymentService$1.run(WebDeploymentService.java:96)
    ... 6 more
```

Não é possível simplesmente por meio de um parâmetro da imagem. Precisamos alterar o arquivo `/opt/eap/standalone/configuration/logging.properties` para configurar o _formatter_ `OPENSHIFT` do log4j.

Visualizar o `/opt/eap/standalone/configuration/logging.properties` e observar a configuração do `formatter.OPENSHIFT`:

```bash
docker run --rm \
  registry.redhat.io/jboss-eap-6/eap64-openshift:1.9 \
  cat -n /opt/eap/standalone/configuration/logging.properties
```

Agora vamos criar um Dockerfile conforme abaixo, que vai configurar o `formatter.OPENSHIFT` para printar o stacktrace no formato que desejamos:

```dockerfile
FROM registry.redhat.io/jboss-eap-6/eap64-openshift:1.9

# backup do arquivo original
RUN cp -a /opt/eap/standalone/configuration/logging.properties \
          /opt/eap/standalone/configuration/logging.properties.original

# adiciona a propriedade exceptionOutputType
RUN sed -i -E \
      's|^formatter.OPENSHIFT.properties=metaData$|formatter.OPENSHIFT.properties=metaData,exceptionOutputType|' \
      /opt/eap/standalone/configuration/logging.properties

# define exceptionOutputType=FORMATTED (default: DETAILED)
RUN echo -e \
      '\nformatter.OPENSHIFT.exceptionOutputType=FORMATTED' >> \
      /opt/eap/standalone/configuration/logging.properties
```

> 📋 As três linhas de `RUN` podem ficar em um único `RUN`. Estão separadas para melhorar a visualização.

Com o Dockerfile já criado, vamos construir a imagem:

```bash
docker build -t jboss:json .
```

Agora é só executar como já vimos anteriormente:

```bash
# executar
docker run --rm \
  --env ENABLE_JSON_LOGGING=true \
  jboss:json

# ou se você quiser ver o JSON formatado
jq -R '. as $raw | try fromjson catch $raw' <(\
  docker run --rm \
    --env ENABLE_JSON_LOGGING=true \
    jboss:json \
)
```

E se houver uma exceção, o log com o stacktrace será algo parecido com:

```json
{
  "@version": 1,
  "@timestamp": "2020-11-21T11:18:28-03:00",
  "sequence": 7086,
  "loggerClassName": "org.jboss.msc.service.ServiceLogger_$logger",
  "loggerName": "org.jboss.msc.service.fail",
  "level": "ERROR",
  "message": "MSC000001: Failed to start service jboss.web.deployment.default-host./xxx",
  "threadName": "ServerService Thread Pool -- 107",
  "threadId": 237,
  "mdc": {},
  "ndc": "",
  "stackTrace": "org.jboss.msc.service.StartException in service jboss.web.deployment.default-host./xxx: org.jboss.msc.service.StartException in anonymous service: JBAS018040: Falha ao iniciar o contexto\n\tat org.jboss.as.web.deployment.WebDeploymentService$1.run(WebDeploymentService.java:99)\n\tat java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)\n\tat java.util.concurrent.FutureTask.run(FutureTask.java:266)\n\tat java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)\n\tat java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)\n\tat java.lang.Thread.run(Thread.java:748)\n\tat org.jboss.threads.JBossThread.run(JBossThread.java:122)\nCaused by: org.jboss.msc.service.StartException in anonymous service: JBAS018040: Falha ao iniciar o contexto\n\tat org.jboss.as.web.deployment.WebDeploymentService.doStart(WebDeploymentService.java:168)\n\tat org.jboss.as.web.deployment.WebDeploymentService.access$000(WebDeploymentService.java:61)\n\tat org.jboss.as.web.deployment.WebDeploymentService$1.run(WebDeploymentService.java:96)\n\t... 6 more\n",
  "log-handler": "CONSOLE"
}
```

> 📋 Agora todo o stacktrace é um único atributo no json.

Uma ferramenta como o Kibana vai mostrar o log acima formatado, algo parecido com:

![Kibana - Stacktrace formatado]({{ site.baseurl }}/assets/img/posts/jboss-eap-docker-image-stacktrace-json-log/kibana-stacktrace.png.jpg)

Ou você pode salvar o log acima para um arquivo e ver formatado, assim:

```bash
while read -r line; do echo -e $line; done <log.json
```

## Referências

- Muito grep dentro da imagem do JBoss
- <https://access.redhat.com/solutions/3318531>
- <https://github.com/jamezp/jboss-logmanager-ext>
- <https://github.com/jamezp/jboss-logmanager-ext/blob/1472e87fffc1a6ea557172d28d598d134d173295/src/main/java/org/jboss/logmanager/ext/formatters/StructuredFormatter.java#L103-L118>
