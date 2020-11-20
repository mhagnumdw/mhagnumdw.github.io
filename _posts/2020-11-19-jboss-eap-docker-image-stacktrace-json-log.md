---
layout: post
title: 'Imagen docker JBoss EAP 6.4: Logs em JSON e formato do stacktrace'
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

Ativar o log no formato JSON da imagem docker do JBoss EAP 6.4. Também vamos mudar o formato que o stacktrace aparece no JSON.

<!--more-->

// TODO: explicar como a imagem do JBoss ativa o log de JSON...

As imagens oficiais podem ser obtidas do docker registry da RedHat: registry.redhat.io

```bash
# listar as tags disponíveis
skopeo list-tags docker://registry.redhat.io/jboss-eap-6/eap64-openshift
```

Para ativar o log no formato JSON do JBoss, basta subir um container setando a variável de ambiente `ENABLE_JSON_LOGGING` para `true`.

A partir disso casa linha do log da aplicação será parecido com isso:

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

Se houver exceção, o log será parecido com isso:

> 📋 Observar que cada linha do stacktrace ficou em um fragmento de JSON, o que pode deixar um pouco chato a visualização.

<details>
  <summary>Clique para visualizar o log em JSON</summary>

```json
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
```

</details>

Podemos configurar para que o stacktrace da exception fique no formato padrão, de quando visualizamos no console.

// TODO: daquiiiii

## Referências

- <...........>
