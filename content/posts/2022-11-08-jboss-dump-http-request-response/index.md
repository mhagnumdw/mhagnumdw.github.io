---
title: 'JBoss EAP 6: dump da requisição http recebida'
date: "2022-11-08T13:26:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
resources:
- name: "featured-image"
  src: "banner.png"
categories: ["JBoss"]
tags: ["jboss", "http"]
---

Logar o request/response das requisições HTTP feitas para o JBoss.

<!--more-->

No `standalone.xml` (ou `domain.xml`, se o JBoss estiver no modo domain), adicionar no substytem `web` o _valve_ `org.apache.catalina.valves.RequestDumperValve`.

### Configuração

```xml
<subsystem xmlns="urn:jboss:domain:web:2.2" default-virtual-server="default-host" native="false">
  <!-- omitido -->
  <valve name="RequestDumperValve" module="org.jboss.as.web"
    class-name="org.apache.catalina.valves.RequestDumperValve"/>
  <!-- omitido -->
</subsystem>
```

### Resultado

O log será algo parecido com isso:

```text
REQUEST URI       =/grpfor/pagesPublic/iptu/damIptu/imprimirDamIptuMobile.seam
          authType=null
 characterEncoding=null
     contentLength=2068
       contentType=application/x-www-form-urlencoded
       contextPath=/grpfor
            cookie=JSESSIONID=xxxxxxxxxxxxxxxxxxxxxxxx
            cookie=PGADMIN_LANGUAGE=en
            cookie=_xsrf=2|d0846c95|ddbb61a88df7175728cc3634b5b0501a|1666977698
            cookie=username-localhost-8888=2|1:0|10:1666979239|23:username-localhost-8888|44:ODdmYzMxOWNmNmUwNGRkNGFmNmE0MDYzMzAyNzJlOWQ=|6295a7b0330bedc12c94d1214aa69f17b2255f2323d8464670281037b7b2fdee
            header=host=localhost:8080
            header=connection=keep-alive
            header=content-length=2068
            header=cache-control=max-age=0
            header=sec-ch-ua="Google Chrome";v="107", "Chromium";v="107", "Not=A?Brand";v="24"
            header=sec-ch-ua-mobile=?0
            header=sec-ch-ua-platform="Linux"
            header=upgrade-insecure-requests=1
            header=origin=http://localhost:8080
            header=content-type=application/x-www-form-urlencoded
            header=user-agent=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
            header=accept=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
            header=sec-fetch-site=same-origin
            header=sec-fetch-mode=navigate
            header=sec-fetch-user=?1
            header=sec-fetch-dest=document
            header=referer=http://localhost:8080/grpfor/pagesPublic/iptu/damIptu/imprimirDamIptuMobile.seam
            header=accept-encoding=gzip, deflate, br
            header=accept-language=pt-BR,pt;q=0.9,en-US;q=0.8,en;q=0.7,und;q=0.6,es;q=0.5
            header=cookie=JSESSIONID=xxxxxxxxxxxxxxxxxxxxxxxx; PGADMIN_LANGUAGE=en; _xsrf=2|d0846c95|ddbb61a88df7175728cc3634b5b0501a|1666977698; username-localhost-8888="2|1:0|10:1666979239|23:username-localhost-8888|44:ODdmYzMxOWNmNmUwNGRkNGFmNmE0MDYzMzAyNzJlOWQ=|6295a7b0330bedc12c94d1214aa69f17b2255f2323d8464670281037b7b2fdee"
            locale=pt_BR
            method=POST
         parameter=pmfInclude:cadastroForm:j_id379=pmfInclude:cadastroForm:j_id379
         parameter=pmfInclude:cadastroForm=pmfInclude:cadastroForm
         parameter=javax.faces.ViewState=j_id1
         parameter=pmfInclude:cadastroForm:dataNascimentoInputCurrentDate=11/2022
         parameter=pmfInclude:cadastroForm:inscricaoImovelDec:j_id60=03AEkXODBVkNpJAWdRF-ERvx3UQK6puS8svLq54AAaolH8g7J-yEslcTYKrM607sx869rtlzZX1Qta89qcdjcmfh2fiNJ_ns3DZ3L2hwcle0UQSU7nQfwPDXqYnN2alZKR833IwPwS3YGbhEqUHflmp0FNB3InIPMlBQi9zv7TwrkFAwvjuGHlxzTTiisNdygekWzyJafqECG6B8oN--CBRJvPZC1OHVaeddUSTIbPKCUVAjaCHBkDHhRXuMXGML-HdROUH3gJJumWJmHw-UpCaF95SM__KMUSonx0Ke92S0qKKUXdTZ017ZpI5xp5sSTrg8oeUm6SQAFYG_tZVsU8pZJMzXLBQW40Al5Fqw0GCdOSZma0JcClLdreHaL76CK1PCFpViqRe_NI5J8HQvfh8taYkFoAygzSR7nAtx9RNyDdJs2xUw_u4dobl1ap_mAUpDKpKtNhEW40EQL5VeoxRBy951-QU7v1Lrt323qxTMpvB0HNDH0Z9lABtYgRxHy4wJkSs0aPDW4tRK_lqxAAO-GweuVZoC1TnRMCyWG3292162BzbVeH3EQ
         parameter=pmfInclude:cadastroForm:j_id84=03AEkXODBVkNpJAWdRF-ERvx3UQK6puS8svLq54AAaolH8g7J-yEslcTYKrM607sx869rtlzZX1Qta89qcdjcmfh2fiNJ_ns3DZ3L2hwcle0UQSU7nQfwPDXqYnN2alZKR833IwPwS3YGbhEqUHflmp0FNB3InIPMlBQi9zv7TwrkFAwvjuGHlxzTTiisNdygekWzyJafqECG6B8oN--CBRJvPZC1OHVaeddUSTIbPKCUVAjaCHBkDHhRXuMXGML-HdROUH3gJJumWJmHw-UpCaF95SM__KMUSonx0Ke92S0qKKUXdTZ017ZpI5xp5sSTrg8oeUm6SQAFYG_tZVsU8pZJMzXLBQW40Al5Fqw0GCdOSZma0JcClLdreHaL76CK1PCFpViqRe_NI5J8HQvfh8taYkFoAygzSR7nAtx9RNyDdJs2xUw_u4dobl1ap_mAUpDKpKtNhEW40EQL5VeoxRBy951-QU7v1Lrt323qxTMpvB0HNDH0Z9lABtYgRxHy4wJkSs0aPDW4tRK_lqxAAO-GweuVZoC1TnRMCyWG3292162BzbVeH3EQ
         parameter=pmfInclude:cadastroForm:j_id29=03AEkXODBVkNpJAWdRF-ERvx3UQK6puS8svLq54AAaolH8g7J-yEslcTYKrM607sx869rtlzZX1Qta89qcdjcmfh2fiNJ_ns3DZ3L2hwcle0UQSU7nQfwPDXqYnN2alZKR833IwPwS3YGbhEqUHflmp0FNB3InIPMlBQi9zv7TwrkFAwvjuGHlxzTTiisNdygekWzyJafqECG6B8oN--CBRJvPZC1OHVaeddUSTIbPKCUVAjaCHBkDHhRXuMXGML-HdROUH3gJJumWJmHw-UpCaF95SM__KMUSonx0Ke92S0qKKUXdTZ017ZpI5xp5sSTrg8oeUm6SQAFYG_tZVsU8pZJMzXLBQW40Al5Fqw0GCdOSZma0JcClLdreHaL76CK1PCFpViqRe_NI5J8HQvfh8taYkFoAygzSR7nAtx9RNyDdJs2xUw_u4dobl1ap_mAUpDKpKtNhEW40EQL5VeoxRBy951-QU7v1Lrt323qxTMpvB0HNDH0Z9lABtYgRxHy4wJkSs0aPDW4tRK_lqxAAO-GweuVZoC1TnRMCyWG3292162BzbVeH3EQ
         parameter=autoScroll=
          pathInfo=null
          protocol=HTTP/1.1
       queryString=null
        remoteAddr=127.0.0.1
        remoteHost=127.0.0.1
        remoteUser=null
requestedSessionId=dPMal-aqW55AO4aaBIdeE0UL
            scheme=http
        serverName=localhost
        serverPort=8080
       servletPath=/pagesPublic/iptu/damIptu/imprimirDamIptuMobile.seam
          isSecure=false
---------------------------------------------------------------
---------------------------------------------------------------
          authType=null
     contentLength=-1
       contentType=application/pdf
            header=X-UA-Compatible=IE=EmulateIE8
            header=X-Powered-By=JSF/1.2
            header=Content-Disposition=attachment; filename="NotificacaoIsencaoImunidade.pdf"
            header=Content-Type=application/pdf
            header=Transfer-Encoding=chunked
            header=Date=Tue, 08 Nov 2022 15:04:42 GMT
           message=null
        remoteUser=null
            status=200
===============================================================
```

### Referências

- [How to enable RequestDumperValve in EAP 6](https://access.redhat.com/solutions/548593)
