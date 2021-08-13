---
layout: post
title: 'JBoss EAP 6.4.0: atualizar para 6.4.20 (patches)'
date: 2018-06-21 10:16:00 -03:00
categories:
- JBoss EAP
tags:
- patches
- update
author-id: mhagnumdw
image: "assets/img/posts/jboss-eap-6-4-0-atualizar-para-6-4-20-patches/jboss-eap-6-4-0-atualizar-para-6-4-20-patches.png"
feature-img: "assets/img/posts/jboss-eap-6-4-0-atualizar-para-6-4-20-patches/jboss-eap-6-4-0-atualizar-para-6-4-20-patches.png"
thumbnail: "assets/img/posts/jboss-eap-6-4-0-atualizar-para-6-4-20-patches/jboss-eap-6-4-0-atualizar-para-6-4-20-patches.png"
---

Aplicar patches ao JBoss EAP 6.4.0 usando o _patch management system_ que foi introduzido a partir do JBoss EAP 6.2.

<!--more-->

Antes de tudo fazer backup da instalação atual do JBoss EAP.

> 📋 Os comandos abaixo foram executados no JBoss no modo standalone, para o modo domain é necessário acrescenter o parâmetro `--host=$HOST` a alguns comandos

## Verificar a instalação atual

```shell
# Conectar ao jboss via cli
./jboss-eap-6.4/bin/jboss-cli.sh -c

# retorna informações sobre a instalação atual
patch info
```

Abaixo o output do comando acima

```json
{
    "outcome" : "success",
    "result" : {
        "cumulative-patch-id" : "base",
        "patches" : [],
        "version" : "6.4.0.GA",
        "addon" : null,
        "layer" : {"base" : {
            "cumulative-patch-id" : "base",
            "patches" : []
        }}
    }
}
```

## Baixar os patches

<https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?product=appplatform&downloadType=patches&version=6.4>

Até a data de hoje (21/06/2018) existem 20 patches disponíveis. Bastando aplicar apenas os patches e em ordem: `jboss-eap-6.4.9-patch.zip`, `jboss-eap-6.4.19-patch.zip` e `jboss-eap-6.4.20-patch.zip`

**O patche 9 traz os patches de 1 a 9, o patche 19 traz os patches de 10 a 19.** [1]

## Instalando

```shell
# Aplicando o patch 9 (1 a 9)
patch apply ~/ambiente/servers/jboss/jboss-eap-6.4-patches/jboss-eap-6.4.9-patch.zip

# Reiniciar o JBoss
shutdown --restart=true

# Aplicando o patch 19 (10 a 19)
patch apply ~/ambiente/servers/jboss/jboss-eap-6.4-patches/jboss-eap-6.4.19-patch.zip

# Reiniciar o JBoss
shutdown --restart=true

# Aplicando o patch 20
patch apply ~/ambiente/servers/jboss/jboss-eap-6.4-patches/jboss-eap-6.4.20-patch.zip

# Reiniciar o JBoss
shutdown --restart=true

# Verificando o resultado
patch history
patch info

# Apagando o histórico de patches
# É mantido um histórico de patches para que possa ser dado rollback
/core-service=patching:ageout-history
```

## Referências

- <https://access.redhat.com/articles/2605021>
- <https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/6.4/html-single/installation_guide/#sect-Patching_a_ZipInstaller_Installation>
