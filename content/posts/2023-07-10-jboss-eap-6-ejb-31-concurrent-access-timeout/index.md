---
title: JBoss EAP 6 EJB 3.1 Concurrent Access Timeout InterceptorContext
date: "2023-07-10T09:00:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
resources:
- name: "featured-image"
  src: "banner.png"
categories: ["JBoss", "EJB"]
tags: ["JBoss EAP", "EJB 3.1", "Transaction"]
---

O seguinte erro ocorre em chamadas EJB: `JBAS014360: EJB 3.1 FR 4.3.14.1 concurrent access timeout on org.jboss.invocation.InterceptorContext@ad0f81 - could not obtain lock within 5000 MILLISECONDS`

<!--more-->

> - Esse post é uma cópia da referência [1]
> - Resolvi pubicar aqui porque é um conteúdo bem antigo que quase não se acha mais na internet

## JBAS014360: EJB 3.1 FR 4.3.14.1 concurrent access timeout on org.jboss.invocation.InterceptorContext@ad0f81 - could not obtain lock within 5000 MILLISECONDS

### Environment

- Red Hat JBoss Enterprise Application Platform(EAP)
  - 6.x
  - 5.x
- Java EE

### Issue

- To understand better the `ConcurrentAccessTimeoutException:` JBAS014360: EJB 3.1 FR 4.3.14.1 concurrent access timeout on org.jboss.invocation.InterceptorContext@ad0f81 - could not obtain lock within 5000 MILLISECONDS exception in logs.

A `StatefulBean` that is being called by a stateless bean.

The scenario is:

1. Call `Stateless Bean`method1()`marked with`@TransactionAttribute(value=TransactionAttributeType.REQUIRED)`
2. `StatelessBean` gets reference to `StatefulBean`
3. `StatelessBean` calls `StateFulBean` method marked with `@TransactionAttribute(value = TransactionAttributeType.REQUIRES_NEW)`.\
    The method which is calling does nothing, it is an empty method
4. This scenario throws the following exception:

```log
2015-01-27 15:50:37,284 ERROR [org.jboss.as.ejb3.invocation] (http-/0.0.0.0:8443-1) JBAS014134: EJB Invocation failed on component TransTestSFBean for method public void ejb.test.TestSFBean.doNothing(): javax.ejb.ConcurrentAccessTimeoutException: JBAS014360: EJB 3.1 FR 4.3.14.1 concurrent access timeout on org.jboss.invocation.InterceptorContext@4375b1 - could not obtain lock within 5000 MILLISECONDS
    at org.jboss.as.ejb3.component.stateful.StatefulSessionSynchronizationInterceptor.processInvocation(StatefulSessionSynchronizationInterceptor.java:117) [jboss-as-ejb3-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    at org.jboss.invocation.InterceptorContext.proceed(InterceptorContext.java:288) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.invocation.InitialInterceptor.processInvocation(InitialInterceptor.java:21) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.invocation.InterceptorContext.proceed(InterceptorContext.java:288) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.invocation.ChainedInterceptor.processInvocation(ChainedInterceptor.java:61) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.as.ee.component.interceptors.ComponentDispatcherInterceptor.processInvocation(ComponentDispatcherInterceptor.java:53) [jboss-as-ee-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    at org.jboss.invocation.InterceptorContext.proceed(InterceptorContext.java:288) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.as.ejb3.component.stateful.StatefulComponentInstanceInterceptor.processInvocation(StatefulComponentInstanceInterceptor.java:67) [jboss-as-ejb3-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    at org.jboss.invocation.InterceptorContext.proceed(InterceptorContext.java:288) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.as.ejb3.tx.CMTTxInterceptor.invokeInOurTx(CMTTxInterceptor.java:272) [jboss-as-ejb3-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    at org.jboss.as.ejb3.tx.CMTTxInterceptor.requiresNew(CMTTxInterceptor.java:363) [jboss-as-ejb3-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    at org.jboss.as.ejb3.tx.CMTTxInterceptor.processInvocation(CMTTxInterceptor.java:240) [jboss-as-ejb3-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    at org.jboss.invocation.InterceptorContext.proceed(InterceptorContext.java:288) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.as.ejb3.component.interceptors.CurrentInvocationContextInterceptor.processInvocation(CurrentInvocationContextInterceptor.java:41) [jboss-as-ejb3-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    at org.jboss.invocation.InterceptorContext.proceed(InterceptorContext.java:288) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.as.ejb3.component.interceptors.ShutDownInterceptorFactory$1.processInvocation(ShutDownInterceptorFactory.java:64) [jboss-as-ejb3-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    at org.jboss.invocation.InterceptorContext.proceed(InterceptorContext.java:288) [jboss-invocation-1.1.2.Final-redhat-1.jar:1.1.2.Final-redhat-1]
    at org.jboss.as.ejb3.component.interceptors.LoggingInterceptor.processInvocation(LoggingInterceptor.java:59) [jboss-as-ejb3-7.3.0.Final-redhat-14.jar:7.3.0.Final-redhat-14]
    ........

```

- is it possible to invoke a Stateful Session Bean several times in a transaction and use methods which are marked with TransactionAttributeType.REQUIRES_NEW?

### Resolution

According to the EJB specification it is not possible to enlist a Stateful Session Bean in more than one transaction at a time.\
The application need to prevent such use-case.

### Root Cause

According to the EJB specification it is only possible to use a StatefulBean in one transaction at a time.

See EJB3.1 specification chapter **4.6.4 Restrictions for transactions**

> _The state diagram implies the following restrictions on transaction scoping of the client invoked business methods. The restrictions are enforced by the container and must be observed by the client programmer._
>
> - _A session bean instance can participate in at most a single transaction at a time._
> - _If a session bean instance is participating in a transaction, it is an error for a client to invoke a method on the session object such that the transaction attribute specified in the bean's metadata annotations and/or the deployment descriptor would cause the container to execute the method in a different transaction context or in an unspecified transaction context. In such a case, the javax.ejb.EJBException will be thrown to a client of the bean's business interface_

## References

- <https://access.redhat.com/solutions/1341983> [1]
