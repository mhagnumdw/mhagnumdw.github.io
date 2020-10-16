---
layout: post
title: 'Jenkins: Renomear Jobs em massa'
date: 2016-09-27 21:56:16 -03:00
categories:
- Jenkins
tags:
- jenkins
author-id: mhagnumdw
image: "assets/img/posts/renomear-jobs-em-massa-no-jenkins/jenkins_script_console_v0.png"
feature-img: "assets/img/posts/renomear-jobs-em-massa-no-jenkins/jenkins_script_console_v0.png"
thumbnail: "assets/img/posts/renomear-jobs-em-massa-no-jenkins/jenkins_script_console_v0.png"
---

O Jenkins não possui uma função nativa para renomear jobs em massa. Quando o número de jobs é elevado fica inviável renomear manualmente.

<!--more-->

## Solução

Para esse exemplo os nomes atuais dos jobs são:

- techthingscool-core-v1
- techthingscool-core-v2
- techthingscool-core-v3

E será removido "-core", ficando:

- techthingscool-v1
- techthingscool-v2
- techthingscool-v3

### Abrir Script Console do Jenkins

Jenkins > Manage Jenkins > Script Console

Colar o script abaixo

```groovy
import hudson.model.*

disableChildren(Hudson.instance.items)

def disableChildren(items) {
    for (item in items) {
        if (( m = item.name =~ /^(techthingscool)-core-(.*)$/)) {
            println("Antigo Nome: " + item.name)
            newname = m.group(1) + "-" + m.group(2)
            println("  Novo Nome: " + newname)
            println("Renomeando...")
            item.renameTo(newname)
            println("Renomeado!")
        }
    }
    println("Fim!")
}
```

Foi utilizado um regex para filtrar apenas o jobs que se deseja renomear. O regex está em vermelho no script acima.

**Clicar em "Run" e  pronto!**

### Sobre o regex do exemplo acima

```text
^(techthingscool)-core-(.*)$
```

- `(techthingscool)` é o grupo 1
- `(.*)` é o grupo 2

## Dica

É bom comentar a linha **item.renameTo(newname)** e rodar o script para checar os jobs que foram capturados pelo regex e os novos nomes.

## Referências

[https://wiki.jenkins-ci.org/display/JENKINS/Bulk+rename+projects](https://wiki.jenkins-ci.org/display/JENKINS/Bulk+rename+projects)
