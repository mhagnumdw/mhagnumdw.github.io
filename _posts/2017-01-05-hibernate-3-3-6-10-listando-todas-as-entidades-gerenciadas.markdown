---
layout: post
title: Hibernate 3 / 3.6.10 - Listando todas as entidades gerenciadas
date: 2017-01-05 14:09:52 -03:00
categories:
- Hibernate 3
tags:
- Hibernate
- Java
- JPA
author: mhagnumdw
feature-img: "assets/test_v3.png"
thumbnail: "assets/test_v3.png"
---

Objetivo: listar todas as entidades gerenciadas pelo EntityManager no Hibernate 3.6.10 (JPA 1)

{% highlight java %}
EntityManager em = ;

// recuperar aqui alguma entidade para vê-la gerenciada no loop abaixo

SessionImplementor session = (SessionImplementor) em.getDelegate();
PersistenceContext persistenceContext = session.getPersistenceContext();
IdentityMap entityEntries = (IdentityMap) persistenceContext.getEntityEntries();
List<IdentityMapEntry> entryList = entityEntries.entryList();
for (IdentityMapEntry ime : entryList) {
    EntityEntry ee = (EntityEntry) ime.getValue();
    Object entidade = ime.getKey();
    System.out.println(ee.getEntityName() + " - " + ee.getId() + " - " + ee.getStatus());
}
{% endhighlight %}

imports:
- org.hibernate.engine.SessionImplementor
- org.hibernate.engine.PersistenceContext
- org.hibernate.util.IdentityMap
- org.hibernate.util.IdentityMap.IdentityMapEntry
- org.hibernate.engine.EntityEntry

Obs: para JPA 2 é diferente