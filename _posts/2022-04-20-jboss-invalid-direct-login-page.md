---
layout: post
title: 'JBWEB000006: Invalid direct reference to form login page'
date: 2022-04-20 11:32:00 -03:00
categories:
- JBoss
tags:
- http
- jboss
- tomcat
- JASS
author-id: mhagnumdw
image: "assets/img/posts/jboss-invalid-direct-login-page/banner.png"
feature-img: "assets/img/posts/jboss-invalid-direct-login-page/banner.png"
thumbnail: "assets/img/posts/jboss-invalid-direct-login-page/banner.png"
---

Como corrigir o problema de 'HTTP Status 400 - JBWEB000006: Invalid direct reference to form login page'.

<!--more-->

Ao acessar a aplicação pela página de login diretamente, como ex <http://localhost:8080/idp/j_security_check>, surge o erro de _Invalid direct reference to form login page_ após o login com sucesso. Ao acessar a aplicação normalmente, algo como <http://localhost:8080/idp>, tudo funciona bem.

## Versões no ambiente

- JBoss EAP 6.4.23

## Motivo

Se você tentar acessar um recurso protegido, o JBoss vai redirecionar para a página de login, conforme configuração no `web.xml`. Uma vez autenticado, ele irá redirecionar para a página inicial solicitada. Se você acessar diretamente a página de login, como ex <http://localhost:8080/idp/j_security_check>, não há solicitação inicial (para onde o usuário desejaria ir), daí o erro: "HTTP Status 400 - JBWEB000006: Invalid direct reference to form login page".

## Log

Ativando os logs `org.apache.catalina` e `org.jboss.security` é possível ver a tentativa de redirect para `null` que faz surgir o problema. Observar no log abaixo a linha **Redirecting to original 'null'**.

```text
DEBUG [org.apache.catalina.authenticator] (http-0.0.0.0:8080-5) Security checking request POST /idp/j_security_check
DEBUG [org.apache.catalina.authenticator] (http-0.0.0.0:8080-5) Authenticating username '11111111111'
TRACE [org.jboss.security] (http-0.0.0.0:8080-5) PBOX000200: Begin isValid, principal: 11111111111, cache entry: org.jboss.security.authentication.JBossCachedAuthenticationManager$DomainInfo@633c863
TRACE [org.jboss.security] (http-0.0.0.0:8080-5) PBOX000204: Begin validateCache, domainInfo: org.jboss.security.authentication.JBossCachedAuthenticationManager$DomainInfo@633c863, credential class: class java.lang.String
TRACE [org.jboss.security] (http-0.0.0.0:8080-5) PBOX000205: End validateCache, result = true
TRACE [org.jboss.security] (http-0.0.0.0:8080-5) PBOX000201: End isValid, result = true
DEBUG [org.apache.catalina.authenticator] (http-0.0.0.0:8080-5) Authentication of '11111111111' was successful
DEBUG [org.apache.catalina.authenticator] (http-0.0.0.0:8080-5) Redirecting to original 'null'
DEBUG [org.apache.catalina.authenticator] (http-0.0.0.0:8080-5)  Failed authenticate() test ??/idp/j_security_check
TRACE [org.jboss.security] (http-0.0.0.0:8080-5) PBOX000354: Setting security roles ThreadLocal: null
```

## Solução

Configurar a página de _landing_, que será utilizada para redirecionar o usuário após o login quando não houver uma requisição inicial para outro recurso.

No `jboss-web.xml` adicionar:

```xml
<valve>
    <class-name>org.apache.catalina.authenticator.FormAuthenticator
    </class-name>
    <param>
        <param-name>landingPage</param-name>
        <param-value>/</param-value>
    </param>
</valve>
```

O `param-value` deve ser ajustado conforme a aplicação. Observar também que no exemplo acima a autenticação é do tipo `form`.

## Referências

- <https://access.redhat.com/solutions/784923>
- <https://stackoverflow.com/questions/24451777/tomcat-jdbc-realm-j-security-not-redirecting#24484258>
