---
author-id: mhagnumdw
categories:
- spring
date: "2021-07-05T15:30:00Z"
feature-img: assets/img/posts/spring-filters/banner.png
image: assets/img/posts/spring-filters/banner.png
tags:
- spring
- filters
thumbnail: assets/img/posts/spring-filters/banner.png
title: 'Spring Boot: filtros que podem ser úteis'
---

Alguns filtros que podem ajudar durante o desenvolvimento com o Spring Boot.

<!--more-->

## Checar o tipo da conexão com relação a segurança

```java
import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

/**
 * Checa o tipo da conexão com relação a segurança.
 */
@Component
@Order(1)
public class CheckConnectionTypeFilter implements Filter {

    private static final Logger log = LoggerFactory.getLogger(CheckConnectionTypeFilter.class);

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        log.info("uri: {}", req.getRequestURI());
        log.info("Seguro (https)? {}", req.isSecure());
        log.info("schema: {}", req.getScheme());
        log.info("X-SSL-Secure: {}", req.getAttribute("X-SSL-Secure"));
        log.info("javax.servlet.http.sslsessionid: {}", req.getAttribute("javax.servlet.http.sslsessionid"));
        log.info("javax.servlet.request.key_size: {}", req.getAttribute("javax.servlet.request.key_size"));
        log.info("javax.servlet.request.X509Certificate: {}", req.getAttribute("javax.servlet.request.X509Certificate"));

        chain.doFilter(request, response);
    }

}
```

* * *

## Interceptar eventos da `Session`

```java
import java.util.concurrent.atomic.AtomicInteger;

import javax.servlet.http.HttpSessionEvent;
import javax.servlet.http.HttpSessionListener;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HttpSessionConfig {

    private static final Logger log = LoggerFactory.getLogger(HttpSessionConfig.class);

    private static final AtomicInteger totalSessions = new AtomicInteger(0);

    // OBS: Com o uso de passivação de sessão, que faz uso da classe JdbcIndexedSessionRepository,
    // esse lister não é chamado. Motivo: a implementação do JdbcIndexedSessionRepository não
    // oferece suporte para eventos de sessão. Isso pode ser visto no JavaDoc da própria classe
    // ou nesse link do Spring:
    // https://docs.spring.io/spring-session/docs/current/reference/html5/#api-jdbcindexedsessionrepository

    /**
     * Personaliza o listener para ouvir eventos referentes a Session.
     */
    @Bean
    public HttpSessionListener httpSessionListener() {
        return new HttpSessionListener() {
            @Override
            public void sessionCreated(HttpSessionEvent hse) {
                log.info("Session adicionada: {}", hse.getSession().getId());
                log.info("Total de sessions: {}", totalSessions.incrementAndGet());
            }

            @Override
            public void sessionDestroyed(HttpSessionEvent hse) {
                log.info("Session removida: {}", hse.getSession().getId());
                log.info("Total de sessions: {}", totalSessions.incrementAndGet());
            }
        };
    }
}
```

* * *

## Dump das requisições web

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.filter.CommonsRequestLoggingFilter;

/**
 * Configurações referentes ao log de requisições web.
 */
@Configuration
public class RequestLoggingFilterConfig {

    /**
     * Configura o log/dump das requisições web.
     * <p>É preciso ativar na configuração de log:
     * {@code logging.level.org.springframework.web.filter.CommonsRequestLoggingFilter=DEBUG}</p>
     */
    @Bean
    public CommonsRequestLoggingFilter logFilter() {
        CommonsRequestLoggingFilter filter = new CommonsRequestLoggingFilter();
        filter.setIncludeQueryString(true);
        filter.setIncludePayload(true);
        filter.setMaxPayloadLength(10000);
        filter.setIncludeHeaders(true);
        filter.setIncludeClientInfo(true);
        // filter.setBeforeMessagePrefix("REQUEST Inicio:");
        // filter.setAfterMessagePrefix("REQUEST Fim:");
        return filter;
    }

}
```
