---
author-id: mhagnumdw
categories:
- Hibernate 3
date: "2017-01-05T14:09:52Z"
feature-img: assets/img/posts/hibernate-3-3-6-10-listando-todas-as-entidades-gerenciadas/hibernate_3_list_all_managed_entities.png
image: assets/img/posts/hibernate-3-3-6-10-listando-todas-as-entidades-gerenciadas/hibernate_3_list_all_managed_entities.png
tags:
- Hibernate
- Java
- JPA
thumbnail: assets/img/posts/hibernate-3-3-6-10-listando-todas-as-entidades-gerenciadas/hibernate_3_list_all_managed_entities.png
title: Hibernate 3 / 3.6.10 - Listando todas as entidades gerenciadas
---

Objetivo: listar todas as entidades gerenciadas pelo EntityManager no Hibernate 3.6.10 (JPA 1)

<!--more-->

```java
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
```

imports:

```java
import org.hibernate.engine.SessionImplementor;
import org.hibernate.engine.PersistenceContext;
import org.hibernate.util.IdentityMap;
import org.hibernate.util.IdentityMap.IdentityMapEntry;
import org.hibernate.engine.EntityEntry;
```

> 📋 Obs: para JPA 2 é diferente
