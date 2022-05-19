---
layout: post
title: 'Openshift - Quem fez o quê? (Audioria)'
date: 2022-05-19 15:42:00 -03:00
categories:
- openshift
tags:
- auditoria
author-id: mhagnumdw
image: "assets/img/posts/openshift-audit-log/banner.png"
feature-img: "assets/img/posts/openshift-audit-log/banner.png"
thumbnail: "assets/img/posts/openshift-audit-log/banner.png"
---

Descobrir quem fez o quê nos recursos do cluster. Por exemplo: quem pausou um `deploymentconfig`?

<!--more-->

## Caso de uso

Se alguém pausar um `dc`, algo como `oc rollout pause dc/app-master` com o nível de log padrão não é possível saber quem fez essa operação.

Vamos ver como é possível rastrear isso.

## Níveis de log

O Openshift possui 4 níveis de log: `Default`, `WriteRequestBodies`, `AllRequestBodies` e `None`. O nível setado por padrão é o `Default`.

Para listar alterações é suficiente definir para `WriteRequestBodies`.

Para mais dealhes sobre cada nível de log ver [aqui](https://docs.openshift.com/container-platform/4.10/security/audit-log-policy-config.html).

## Alterando o nível de log

Basta ditar o `APIServer` e alterar o valor do path `spec.audit.profile` para `WriteRequestBodies`.

```bash
oc edit apiserver cluster
```

Salvar e sair, em seguida a configuração começará a ser aplicada e isso pode levar um tempo. Para monitorar que nós do cluster já estão com a nova configuração, executar:

```bash
oc get kubeapiserver -o=jsonpath='{range .items[0].status.conditions[?(@.type=="NodeInstallerProgressing")]}{.reason}{"\n"}{.message}{"\n"}'
```

## Pesquisando nos logs

São três tipos de log, **OpenShift API server logs**, **Kubernetes API server logs** e **OpenShift OAuth API server logs**.

O local deles pode ser cosultado assim, respectivamente:

```bash
# OpenShift API server logs
oc adm node-logs --role=master --path=openshift-apiserver/

# Kubernetes API server logs
oc adm node-logs --role=master --path=kube-apiserver/

# OpenShift OAuth API server logs
oc adm node-logs --role=master --path=oauth-apiserver/
```

Os arquivos de log podem ser visualizados assim, respectivamente:

```bash
# OpenShift API server logs
oc adm node-logs <node_name> --path=openshift-apiserver/<log_name>

# Kubernetes API server logs
oc adm node-logs <node_name> --path=kube-apiserver/<log_name>

# OpenShift OAuth API server logs
oc adm node-logs <node_name> --path=oauth-apiserver/<log_name>
```

### Cenário de teste

Vamos pesquisar nos logs. Imagine que:

- Alguém pausou um `dc`, ex: `oc rollout pause dc/app-master`
- Existem 3 nós do Openshift de acordo com o comando executado anteriormente, esse: `oc adm node-logs --role=master --path=xxx`

Basta pesquisar em todos os `audit.log`. No caso vamos pesquisar pela palavra **pause**. Seria algo como:

```bash
oc adm node-logs control-plane-0 --path=openshift-apiserver/audit.log | grep -i pause; \
oc adm node-logs control-plane-1 --path=openshift-apiserver/audit.log | grep -i pause; \
oc adm node-logs control-plane-2 --path=openshift-apiserver/audit.log | grep -i pause; \
oc adm node-logs control-plane-0 --path=kube-apiserver/audit.log | grep -i pause; \
oc adm node-logs control-plane-1 --path=kube-apiserver/audit.log | grep -i pause; \
oc adm node-logs control-plane-2 --path=kube-apiserver/audit.log | grep -i pause; \
oc adm node-logs control-plane-0 --path=oauth-apiserver/audit.log | grep -i pause; \
oc adm node-logs control-plane-1 --path=oauth-apiserver/audit.log | grep -i pause; \
oc adm node-logs control-plane-2 --path=oauth-apiserver/audit.log | grep -i pause
```

E então teremos um json que represena a linha de log, com informações como:

- ip que fez a alteração
- que recurso foi alerado
- o que do recurso foi alterado
- quem alterou
- horário da alteração
- etc

Abaixo um exemplo de log:

![Log parte 1]({{ site.baseurl }}/assets/img/posts/openshift-audit-log/log-json-1.png)

![Log parte 2]({{ site.baseurl }}/assets/img/posts/openshift-audit-log/log-json-2.png)

## Notas / Dicas 📋 ⚠️

- Aumentar a verbosidade do log também faz consumir mais largura de banda de rede, disco e CPU.
- Configure para que os logs de auditoria possam ser consumidos pelo Kibana, facilitando assim a consulta.

## Referências

- <https://docs.openshift.com/container-platform/4.10/security/audit-log-policy-config.html>
- <https://docs.openshift.com/container-platform/4.10/security/audit-log-view.html>
- <https://kubernetes.io/docs/tasks/debug/debug-cluster/audit>
