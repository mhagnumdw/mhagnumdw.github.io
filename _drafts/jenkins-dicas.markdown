---
layout: post
title: 'jenkins: dicas'
date: 2019-10-21 08:00:00 -03:00
categories:
- Jenkins
tags:
- jenkins
- dicas
author-id: mhagnumdw
image: "assets/img/posts/jenkins-dicas/xxx.png"
feature-img: "assets/img/posts/jenkins-dicas/xxx.png"
thumbnail: "assets/img/posts/jenkins-dicas/xxx.png"
---

Dicas sobre o Jenkins. Snippets, configurações simples, scripts groovy para autoamizar tarefas etc.

<!--more-->

### Deletar jobs em massa filtando por nome

```groovy
import jenkins.model.*

def matchedJobs = Jenkins.instance.items.findAll { job ->
  job.name =~ /regex_filter_here/
}

matchedJobs.each { job ->
    // println job.name
    job.delete()
}
```

* * *

### Deletar jobs em massa (TODOS, sem filtro!)

> CUIDADO!

```groovy
import jenkins.model.*

def allJobs = Jenkins.instance.items.findAll()

allJobs.each { job ->
  job.delete()
}
```

* * *

### Cancelar todos os jobs em execução

```groovy
import hudson.model.*

def q = Jenkins.instance.queue

q.items.each { q.cancel(it.task) }
```

* * *
