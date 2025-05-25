---
author-id: mhagnumdw
categories:
- Jenkins
date: "2022-11-07T17:26:00Z"
feature-img: assets/img/posts/jenkins-dicas/banner.png
image: assets/img/posts/jenkins-dicas/banner.png
tags:
- jenkins
- dicas
thumbnail: assets/img/posts/jenkins-dicas/banner.png
title: 'Jenkins: Dicas'
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

### Definir um usuário padrão

```groovy
import jenkins.model.*
import hudson.security.*

println "--> creating admin user"

def adminUsername = System.getenv("JENKINS_USER")
def adminPassword = System.getenv("JENKINS_PASS")
assert adminPassword != null : "No JENKINS_USER env var provided, but required"
assert adminPassword != null : "No JENKINS_PASS env var provided, but required"

def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount(adminUsername, adminPassword)
Jenkins.instance.setSecurityRealm(hudsonRealm)

def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
strategy.setAllowAnonymousRead(false)
Jenkins.instance.setAuthorizationStrategy(strategy)

Jenkins.instance.save()
```

* * *

### Alterar o número de executors

```groovy
import jenkins.model.*

Jenkins.instance.setNumExecutors(10)
```

* * *

### Definir o _Crumb Issuer_

```groovy
// imports
import jenkins.model.Jenkins
import hudson.security.csrf.DefaultCrumbIssuer

// get Jenkins instance
Jenkins jenkins = Jenkins.getInstance()

// set default crumb issuer
jenkins.setCrumbIssuer(new DefaultCrumbIssuer(true))

// save current Jenkins state to disk
jenkins.save()
```

* * *

### Definir o global _Quiet Period_

```groovy
import jenkins.model.Jenkins

Jenkins instance = Jenkins.getInstance()

instance.setQuietPeriod(0)
```

* * *
