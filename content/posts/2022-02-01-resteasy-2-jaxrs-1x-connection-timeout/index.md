---
title: RestEasy 2 - Configurar timeout da conexão
date: "2022-02-01T19:49:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
resources:
- name: "featured-image"
  src: "banner.png"
categories: ["RestEasy"]
tags: ["http", "timeout", "jboss"]
---

Configurar o timout de conexão, que por padrão é infinito.

<!--more-->

Lembrando que o RestEasy 2 é baseado no JAX-RS 1.1, enquanto o RestEasy 3 é baseado no JAX-RS 2.0. **O que veremos aqui se aplica ao RestEasy 2**.

## Versões no ambiente

- JBoss EAP 6.4.23
- RestEasy 2.3.10.Final-redhat-1
- httpclient-4.3.6.redhat-1

## RequestConfig

Primeiro é preciso criar uma instância de `RequestConfig` contendo os timeouts. São 3 timeouts descritos abaixo. No exemplo são 20 segundos de timeout para cada tipo. Pode criar uma instância por aplicação.

```java
private static final RequestConfig requestConfig = RequestConfig.custom()
    // (http.connection.timeout) - the time to establish the connection with the remote host
    .setConnectTimeout(20 * 1000)
    // (http.connection-manager.timeout) - the time to wait for a connection from the connection manager/pool
    .setConnectionRequestTimeout(20 * 1000)
    // (http.socket.timeout) - the time waiting for data - after establishing the connection; maximum time of inactivity between two data packets
    .setSocketTimeout(20 * 1000)
    .build();
```

## Criar o cliente http

Criar uma instância do cliente http. Pode criar uma instância por aplicação.

```java
private static final CloseableHttpClient httpClient = HttpClientBuilder
    .create()
    .setDefaultRequestConfig(requestConfig)
    .build();
```

## Criar ums intância de `ApacheHttpClient4Executor` (`ClientExecutor`)

Criar uma instância de `ApacheHttpClient4Executor`.

```java
ApacheHttpClient4Executor clientExecutor = new ApacheHttpClient4Executor(httpClient) {
    @Override
    public void loadHttpMethod(ClientRequest request, HttpRequestBase httpMethod) throws Exception {
        httpMethod.setConfig(requestConfig);
        super.loadHttpMethod(request, httpMethod);
    }
};
```

> 📋 Mais abaixo será explicado porque criar essa classe anônima. E aqui é o pulo do gato! 😺

## Executar a requisição http

Algo como:

```java
RegisterBuiltin.register(ResteasyProviderFactory.getInstance());

// Comentado de propósito, será usado o que foi criado na etapa anterior (ver detalhes mais abaixo)
// ApacheHttpClient4Executor clientExecutor = new ApacheHttpClient4Executor(httpClient);

GoogleRecaptchaV3Client client = ProxyFactory.create(GoogleRecaptchaV3Client.class, base, clientExecutor);

Response response = client.siteverify(...); // executa a requisição http

entity = response.getEntity()
```

> 📋 `GoogleRecaptchaV3Client` é apenas uma interface com o contrato de acesso ao endpoint.

🎉 🎉 Agora o timeout da requisição irá funcionar. Basta acessar um endpoint que demora a responder mais que os timeouts estabelecidos. 🥳 🥳

## Porque definir a classe anônima `ApacheHttpClient4Executor` ?

Foi necessário instânciar a classe anônima `ApacheHttpClient4Executor` sobrescrevendo o método `loadHttpMethod`.

O motivo é porque a configuração de `RequestConfig` (que seta os timeouts), mesmo sendo definida no `CloseableHttpClient` (como visto acima), ou mesmo definida usando o `HttpClientContext` como no snippet abaixo,

```java
// ...
HttpClientContext localContext = HttpClientContext.create();
localContext.setRequestConfig(requestConfig);
clientExecutor.setHttpContext(localContext);
// ...
```

não funciona. Debugando é possível ver que isso ocorre porque no método `org.apache.http.impl.client.InternalHttpClient.doExecute(HttpHost, HttpRequest, HttpContext)` na linha `config = HttpClientParamConfig.getRequestConfig(params);` e `localcontext.setRequestConfig(config);` é definido uma instância de `RequestConfig` **que não é a que definimos**.

Quando sobrescrevemos o método `loadHttpMethod`, a nossa configuração de `RequestConfig` vai prevalecer devido a linha `config = ((Configurable) request).getConfig();` no método `org.apache.http.impl.client.InternalHttpClient.doExecute(HttpHost, HttpRequest, HttpContext)`.

## Referências

- <https://docs.jboss.org/resteasy/docs/2.3.7.Final/userguide/html_single>
- <https://stackoverflow.com/questions/35271668/cannot-set-timeout-on-resteasy-client-on-jboss#48178491>
- <https://www.baeldung.com/httpclient-timeout>
