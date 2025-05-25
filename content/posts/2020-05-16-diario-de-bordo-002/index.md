---
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"

categories:
- Diário de Bordo
date: "2020-05-16T16:14:00Z"

image: assets/img/cockpit-banner.jpg
tags:
- docker
- openshift
- gitlab
- facelets
- jsf

title: 'Diário de Bordo #002 - Ambientes iguais com comportamentos diferentes'
---

Build, pipeline no GitLab, geração de imagem docker de aplicação com jsf 1.2 + Facelets + JBoss EAP 6.4, deploy no OpenShift, ambientes totalmente iguais e comportamentos diferentes ⏩ Troubleshooting

<!--more-->

{% include about_diario_de_bordo.markdown %}

> **Spoiler:** é claro que existia diferença nos ambientes!

## História

Tenho - na prática mesmo não é meu, então só modo de falar - um pipeline montado de uma aplicação Java que gera um `EAR`. Dentro do `EAR` existe um `WAR` e alguns `JARs`.

O pipeline basicamente executa em sequência: compilação, testes, geração do EAR, geração da imagem docker por meio do [s2i (source-to-image)](https://github.com/openshift/source-to-image) e por fim deploy dessa imagem docker no OpenShift.

> Embora o s2i sugira *código para imagem*, estamos utilizando ele para, a partir de um binário compilado, no caso o `EAR`, montar a imagem docker da aplicação.

No nosso cenário cada branch no Git gera automaticamente uma aplicação no OpenShift.

A partir daqui vou chamar as aplicações de `app-master` e `app-branch`.

Ao acessar ambas as aplicações, `app-master` e `app-branch`, e mandar pesquisar em uma tela de pesquisa simples, nada ocorria na tela, ao mandar pesquisa novamente nada ocorria e ao mandar pesquisa pela terceira vez a aplicação quebrava com a stack abaixo:

<details>
  <summary>Stacktrace - Clique para ver (não é relevante)</summary>

```stacktrace
16:46:35,717 SEVERE [javax.enterprise.resource.webcontainer.jsf.lifecycle] (http-10.129.2.161:8080-6) JSF1054: (Phase ID: PROCESS_VALIDATIONS 3, View ID: /pages/cidade/pesquisa.xhtml) Exception thrown during phase execution: javax.faces.event.PhaseEvent[source=com.sun.faces.lifecycle.LifecycleImpl@47d599aa]
16:46:35,719 ERROR [org.jboss.seam.exception.Exceptions] (http-10.129.2.161:8080-6) handled and logged exception: javax.servlet.ServletException: For input string: "org.jboss.seam.ui.NoSelectionConverter.noSelectionValue"
        at javax.faces.webapp.FacesServlet.service(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:295) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:214) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:83) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.web.IdentityFilter.doFilter(IdentityFilter.java:40) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:69) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.web.LoggingFilter.doFilter(LoggingFilter.java:60) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:69) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:73) [jboss-seam.jar:2.2.6.EAP5]
        at com.myapp.sso.SamlSSOObserver.processSamlAuth(SamlSSOObserver.java:157) [grpfor-core.jar:]
        at com.myapp.sso.SamlSSOObserver.doFilter(SamlSSOObserver.java:135) [grpfor-core.jar:]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:69) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.web.MultipartFilter.doFilter(MultipartFilter.java:90) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:69) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.web.ExceptionFilter.doFilter(ExceptionFilter.java:64) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:69) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.web.RedirectFilter.doFilter(RedirectFilter.java:45) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:69) [jboss-seam.jar:2.2.6.EAP5]
        at org.ajax4jsf.webapp.BaseFilter.doFilter(BaseFilter.java:530) [richfaces-impl-3.3.4.Final.SEFINv1.jar:3.3.4.Final.SEFINv1]
        at org.jboss.seam.web.Ajax4jsfFilter.doFilter(Ajax4jsfFilter.java:56) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.servlet.SeamFilter$FilterChainImpl.doFilter(SeamFilter.java:69) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.servlet.SeamFilter.doFilter(SeamFilter.java:158) [jboss-seam.jar:2.2.6.EAP5]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:246) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:214) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.ajax4jsf.webapp.BaseXMLFilter.doXmlFilter(BaseXMLFilter.java:206) [richfaces-impl-3.3.4.Final.SEFINv1.jar:3.3.4.Final.SEFINv1]
        at org.ajax4jsf.webapp.BaseFilter.handleRequest(BaseFilter.java:290) [richfaces-impl-3.3.4.Final.SEFINv1.jar:3.3.4.Final.SEFINv1]
        at org.ajax4jsf.webapp.BaseFilter.processUploadsAndHandleRequest(BaseFilter.java:388) [richfaces-impl-3.3.4.Final.SEFINv1.jar:3.3.4.Final.SEFINv1]
        at org.ajax4jsf.webapp.BaseFilter.doFilter(BaseFilter.java:515) [richfaces-impl-3.3.4.Final.SEFINv1.jar:3.3.4.Final.SEFINv1]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:246) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:214) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at com.myapp.FiltroCorrecaoIE.doFilter(FiltroCorrecaoIE.java:28) [grpfor.jar:]
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:246) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:214) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:231) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:149) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.jboss.as.web.security.SubjectInfoSetupValve.invoke(SubjectInfoSetupValve.java:34) [jboss-as-web-7.5.20.Final-redhat-1.jar:7.5.20.Final-redhat-1]
        at org.jboss.as.jpa.interceptor.WebNonTxEmCloserValve.invoke(WebNonTxEmCloserValve.java:50) [jboss-as-jpa-7.5.20.Final-redhat-1.jar:7.5.20.Final-redhat-1]
        at org.jboss.as.jpa.interceptor.WebNonTxEmCloserValve.invoke(WebNonTxEmCloserValve.java:50) [jboss-as-jpa-7.5.20.Final-redhat-1.jar:7.5.20.Final-redhat-1]
        at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:512) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.jboss.as.web.security.SecurityContextAssociationValve.invoke(SecurityContextAssociationValve.java:169) [jboss-as-web-7.5.20.Final-redhat-1.jar:7.5.20.Final-redhat-1]
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:151) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:97) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.valves.AccessLogValve.invoke(AccessLogValve.java:560) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:102) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:343) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.coyote.http11.Http11Processor.process(Http11Processor.java:856) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.coyote.http11.Http11Protocol$Http11ConnectionHandler.process(Http11Protocol.java:656) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at org.apache.tomcat.util.net.JIoEndpoint$Worker.run(JIoEndpoint.java:926) [jbossweb-7.5.28.Final-redhat-1.jar:7.5.28.Final-redhat-1]
        at java.lang.Thread.run(Thread.java:748) [rt.jar:1.8.0_242]
Caused by: java.lang.NumberFormatException: For input string: "org.jboss.seam.ui.NoSelectionConverter.noSelectionValue"
        at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65) [rt.jar:1.8.0_242]
        at java.lang.Integer.parseInt(Integer.java:580) [rt.jar:1.8.0_242]
        at java.lang.Integer.&lt;init&gt;(Integer.java:867) [rt.jar:1.8.0_242]
        at org.jboss.seam.ui.EntityIdentifierStore.get(EntityIdentifierStore.java:46) [jboss-seam-ui-2.2.6.EAP5.jar:2.2.6.EAP5]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) [rt.jar:1.8.0_242]
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) [rt.jar:1.8.0_242]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) [rt.jar:1.8.0_242]
        at java.lang.reflect.Method.invoke(Method.java:498) [rt.jar:1.8.0_242]
        at org.jboss.seam.util.Reflections.invoke(Reflections.java:22) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.RootInvocationContext.proceed(RootInvocationContext.java:32) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.SeamInvocationContext.proceed(SeamInvocationContext.java:56) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.transaction.RollbackInterceptor.aroundInvoke(RollbackInterceptor.java:28) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.SeamInvocationContext.proceed(SeamInvocationContext.java:68) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.core.MethodContextInterceptor.aroundInvoke(MethodContextInterceptor.java:44) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.SeamInvocationContext.proceed(SeamInvocationContext.java:68) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.core.SynchronizationInterceptor.aroundInvoke(SynchronizationInterceptor.java:32) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.SeamInvocationContext.proceed(SeamInvocationContext.java:68) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.RootInterceptor.invoke(RootInterceptor.java:107) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.JavaBeanInterceptor.interceptInvocation(JavaBeanInterceptor.java:185) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.JavaBeanInterceptor.invoke(JavaBeanInterceptor.java:103) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.ui.EntityIdentifierStore_$$_javassist_seam_34.get(EntityIdentifierStore_$$_javassist_seam_34.java) [jboss-seam-ui-2.2.6.EAP5.jar:2.2.6.EAP5]
        at org.jboss.seam.ui.AbstractEntityLoader.get(AbstractEntityLoader.java:27) [jboss-seam-ui-2.2.6.EAP5.jar:2.2.6.EAP5]
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) [rt.jar:1.8.0_242]
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) [rt.jar:1.8.0_242]
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) [rt.jar:1.8.0_242]
        at java.lang.reflect.Method.invoke(Method.java:498) [rt.jar:1.8.0_242]
        at org.jboss.seam.util.Reflections.invoke(Reflections.java:22) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.RootInvocationContext.proceed(RootInvocationContext.java:32) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.SeamInvocationContext.proceed(SeamInvocationContext.java:56) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.transaction.RollbackInterceptor.aroundInvoke(RollbackInterceptor.java:28) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.SeamInvocationContext.proceed(SeamInvocationContext.java:68) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.transaction.TransactionInterceptor$1.work(TransactionInterceptor.java:97) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.util.Work.workInTransaction(Work.java:61) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.transaction.TransactionInterceptor.aroundInvoke(TransactionInterceptor.java:91) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.SeamInvocationContext.proceed(SeamInvocationContext.java:68) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.core.MethodContextInterceptor.aroundInvoke(MethodContextInterceptor.java:44) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.SeamInvocationContext.proceed(SeamInvocationContext.java:68) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.RootInterceptor.invoke(RootInterceptor.java:107) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.JavaBeanInterceptor.interceptInvocation(JavaBeanInterceptor.java:185) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.intercept.JavaBeanInterceptor.invoke(JavaBeanInterceptor.java:103) [jboss-seam.jar:2.2.6.EAP5]
        at org.jboss.seam.ui.JpaEntityLoader_$$_javassist_seam_33.get(JpaEntityLoader_$$_javassist_seam_33.java) [jboss-seam-ui-2.2.6.EAP5.jar:2.2.6.EAP5]
        at org.jboss.seam.ui.EntityConverter.getAsObject(EntityConverter.java:76) [jboss-seam-ui-2.2.6.EAP5.jar:2.2.6.EAP5]
        at com.sun.faces.renderkit.html_basic.HtmlBasicInputRenderer.getConvertedValue(Unknown Source) [jsf-impl-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at com.sun.faces.renderkit.html_basic.MenuRenderer.convertSelectOneValue(Unknown Source) [jsf-impl-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at com.sun.faces.renderkit.html_basic.MenuRenderer.getConvertedValue(Unknown Source) [jsf-impl-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIInput.getConvertedValue(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIInput.validate(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIInput.executeValidate(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIInput.processValidators(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIComponentBase.processValidators(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIComponentBase.processValidators(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIComponentBase.processValidators(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIComponentBase.processValidators(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at javax.faces.component.UIComponentBase.processValidators(Unknown Source) [jsf-api-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at org.ajax4jsf.component.UIAjaxForm.processValidators(UIAjaxForm.java:82) [richfaces-ui-3.3.4.Final.jar:3.3.4.Final]
        at org.ajax4jsf.component.AjaxViewRoot$3.invokeContextCallback(AjaxViewRoot.java:447) [richfaces-impl-3.3.4.Final.SEFINv1.jar:3.3.4.Final]
        at org.ajax4jsf.component.AjaxViewRoot.processPhase(AjaxViewRoot.java:240) [richfaces-impl-3.3.4.Final.SEFINv1.jar:3.3.4.Final]
        at org.ajax4jsf.component.AjaxViewRoot.processValidators(AjaxViewRoot.java:463) [richfaces-impl-3.3.4.Final.SEFINv1.jar:3.3.4.Final]
        at com.sun.faces.lifecycle.ProcessValidationsPhase.execute(Unknown Source) [jsf-impl-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at com.sun.faces.lifecycle.Phase.doPhase(Unknown Source) [jsf-impl-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        at com.sun.faces.lifecycle.LifecycleImpl.execute(Unknown Source) [jsf-impl-1.2.15.b01-SP2-redhat-1.jar:1.2.15.b01-SP2-redhat-1]
        ... 49 more
```

</details><br/>

Resumindo o stack, o erro era: `Caused by: java.lang.NumberFormatException: For input string: "org.jboss.seam.ui.NoSelectionConverter.noSelectionValue"`

A string `"org.jboss.seam.ui.NoSelectionConverter.noSelectionValue"` era submetida por um combobox quando nenhum valor era selecionado. E ok, esse é o valor esperado.

O stack acima nem é tão relevante.

Então eu fui debugar o código e para minha frustração as classes do `jsf` e `facelets` foram compiladas sem informação de debug. É um parto debugar assim. Pelo debug não consegui ver o problema e eu acabava perdendo tempo demais por ter que debugar "no escuro", então deixei isso de lado, embora eu tenha até chegado a fazer debug remoto, informando para a `JVM`: `-agentlib:jdwp=transport=dt_socket,server=y,address=8000,suspend=n`.

Eu também observei no log, na primeira vez que mandava pesquisar, diversas mensagens como essa:

`19:17:15,290 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/template.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50`

<details>
  <summary>Mais dessas mensagens aqui - Clique para ver</summary>

```log
19:17:15,290 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/template.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,293 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/cabecalho.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:49
19:17:15,293 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/cabecalho.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:49
19:17:15,297 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,297 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,297 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,297 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,297 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,297 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,297 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,297 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,298 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/menu.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,386 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/status.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,386 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/status.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,386 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/status.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,387 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/status.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,387 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/status.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,387 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/status.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,387 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/status.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,389 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/mensagens.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,393 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/toolBarPesquisa.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,396 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/editColuna.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,397 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/editColuna.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,397 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/editColuna.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,397 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/editColuna.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,401 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/label.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,401 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/label.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,401 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/label.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,402 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/label.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,402 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/label.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,402 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/label.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,405 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/footer.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
19:17:15,405 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/WEB-INF/facelets/tags/footer.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50
```

</details><br/>

> **Spoiler:** se eu tivesse dado atenção a essas mensagens acima e se minha memória tivesse funcionado, eu teria matado a charada sem ter perdido muito tempo.

**Mas porque nos ambientes do OpenShift essa simples pesquisa não funcionava e em ambientes que já existiam funcionava?**

Na imagem docker executada no Openshift (onte tínhamos problemas):

- GitLab 12.9 + GitLab Runner 13
- Imagem do agente do GitLab (onde o build da aplicação é feito): Openjdk 1.8.0_252 e Apache Maven 3.6.3
- Runtime: Openjdk 1.8.0_252
- SO RHEL 7 + JBoss EAP 6.4.20

No ambiente que já funcionava (sem problemas):

- Jenkins
- Apache Maven 3.2.1 + Oracle JDK 1.8.0_171
- Runtime: Oracle JDK 1.8.0_171
- SO RHEL 7 + JBoss EAP 6.4.20

Diferenças existiam, então executei a aplicação na minha máquina local por meio da imagem docker e também tive o mesmo problema da exeução no OpenShift.

Verifiquei se o código no ambiente com problema estava o mesmo do ambiente que funcionava e estava.

Baixei o fonte da aplicação, subi no meu Eclipse + JBoss local e não dava problema. No meu ambeinte local eu usava `Oracle JDK 1.8.0_171` e resolvi usar `Openjdk 1.8.0_252` (a versão do container docker, que dava problema), e continuou sem dar problema.

Nessa altura do campeonado eu já tinha feito quase tudo. Já tinha checado versão de SO, versão de `JVM`, versão de JBoss, versão dos módulos do JBoss, versão de código (inclusive o código estava binariamente igual), locale, timezone, configurações de `JVM`, configurações do JBoss, system properties, tudo... ou quase tudo...!

Para piorar, uma das aplicações que citei lá em cima, a aplicação `app-branch`, passou a funcionar, do nada! **Nesse momento `app-branch` e `app-master` estavam iguais (compiladas a partir do mesmo código fonte).** E daí eu comecei a perceber uma alternância de hora funcionar, hora não. Existia um padrão, mas até então eu não tinha observado.

Pensando... pensando... pensando... resolvi comparar o `EAR` gerado pelo Jenkins com o `EAR` gerado pelo pipeline do GitLab, e tinha diferença! Me animei por um tempo, mas quando vi as diferenças, desanimei, eram:

- o arquivo `META-INF\maven\com.myapp\myapp-ear\pom.properties`
- o arquivo `META-INF\MANIFEST.MF`

E a única diferença entre eles eram o usuário do build, horário do build e versão da JDK.

> Nesse momento a versão JDK diferente foi irrelevante, pois eu já sabia disso e já havia feito testes com a mesma JDK usada no container docker.

Subi o `EAR` gerado pelo Jenkins no meu JBoss local (já nem lembro que JDK usei, eu fiz muita coisa!, rs), e a aplicação funcionou ok. Daí resolvi subir também no meu JBoss local o `EAR` gerado pelo pipeline do GitLab, e, para minha surpresa e alegria, deu problema no meu ambiente local!

**Então aqui ficou claro para mim uma coisa muito importante: o problema era no `EAR`! O `EAR` do Jenkins dava certo, o `EAR` do GitLab não!**

Eu olhava para o `EAR` e não via nada de diferente, exceto o que falei acima. Então, já apelando para o impossível, deixei o `EAR` com problema no JBoss e alterei nele apenas o `pom.properties` e o `MANIFEST.MF` para ficar igual ao do `EAR` que dava certo... e... continuou dando problema!

**E agora? Não tinha mais nada diferente. Os `EAR` estavam binariamente iguais. Mas um dava certo e o outro não. E no mesmo ambiente! Sinistro!**

Nesse testes locais eu vi novamente a mensagem que não dei importância, que citei lá em cima, essa:

`19:17:15,290 INFO  [facelets.facelet] (http-0.0.0.0:8080-6) Facelet[/layout/template.xhtml] was modified @ 19:17:15, flushing component applied @ 19:16:50`

> 💭 Porque eu não olhei para isso antes! 💭

Agora eu resolvi dar importância, então dá-lhe Google, e tome logo no primeiro link, no Stackoverflow, mas não na primeira resposta, [mas nessa aqui](https://stackoverflow.com/questions/3820111/facelet-was-modified-messages#10311517).

Bom... lendo o post, o cara fala do atributo `facelets.REFRESH_PERIOD` e quando eu bati o olho nisso lembrei de umas coisas. Então fui checar as datas de modificação dos arquivos `xhtml` em ambos os `EAR`. **No `EAR` problemático a data de todos eles estava no futuro!**

Na primeira requisição, quando esses `xhtml` eram acessados, o `facelets` entedia que eles haviam mudado e os recompilava, o que acaba por recriar a árvore de componentes, o que acabava por perder os dados do request e fazer surgir o problema.

Informei para o `facelets` não checar as mudanças nos `xhtml`, adicionando ao `web.xml`:

```xml
<context-param>
    <param-name>facelets.REFRESH_PERIOD</param-name>
    <param-value>-1</param-value> <!-- padrão: 2 segundos -->
</context-param>
```

E o prolema foi embora! 🎉🎉🎉

**Mas porque a data de modificação dos `xhtml` estava no futuro?** Porque o runner do GitLab estava em `UTC`, quando deveria estar em `UTC-3`. Então, se na verdade eram 20:23:56, para o runner eram 23:23:56 (3h na frente).

> Runner é um container docker, iniciado pelo GitLab, que possui as ferramentas para buildar a aplicação. O build da aplicação, no caso aqui geração do EAR, é feito dentro desse container.

**Porque a aplicação `app-branch` passou a funcionar do nada?** Na verdade não foi do nada. Quando a hora do container docker ficou na frente da hora de modicação dos arquivos `xhtml`, o `facelets` deixou de re-compilar os `xhtml` por entender que eles não foram modificados.

**🧩 Dica:** em produção o parâmetro `facelets.REFRESH_PERIOD` deve ser `-1`, já que os `xhtml` não vão mudar, mas em desenvolvimento, com _hot deploy_, o interessante é que seja `0`, para que as mudanças feitas no código surtam efeito o quanto antes.

Até a próxima!

_ps: é claro que a hora do runner tem que ser ajustada!_

## Referências

- <https://stackoverflow.com/questions/3820111/facelet-was-modified-messages#10311517>
- <https://docs.jboss.org/jbossas/6/JSF_Guide/en-US/html/jsf.reference.html>
