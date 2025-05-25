---
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"

categories:
- deploy
date: "2023-01-18T12:07:44Z"

resources:
- name: "featured-image"
  src: "banner.png"
tags:
- argocd
- gitops
- openshift
- kubernetes

title: ArgoCD CLI - Criar um projeto e adicionar repositórios
---

Vamos configurar um projeto no ArgoCD (por padrão já existe o projeto chamado `default`) e adicionar repositórios. O ArgoCD estará rodando dentro do Openshift.

<!--more-->

Nesse nosso cenário o ArgoCD roda em um namespace e gerencia aplicações de outro namespace. Vamos usar o CLI do ArgoCD, mas tudo pode ser feito pela sua interface web. O repositório de código fica no GitLab.

## Premissas

- ArgoCD instalado no Openshift por meio do OperatorHub (loja de apps do Openshift)
- Nome da instância do ArgoCD: `argo` (vários recursos tem seu nome baseado nesse nome)
- Namespace onde fica o ArgoCD: `openshift-gitops`
- Namespace da aplicação de exemplo: `jarvis`

## Adicionar o client via [asdf](https://github.com/asdf-vm/asdf)

```bash
asdf plugin add argocd
asdf install argocd 2.3.9
asdf global argocd 2.3.9 # ou asdf local argocd 2.3.9 (entenda a diferença entre global e local)
```

## Logar no ArgoCD com usuário `admin`

```bash
argocd login \
  $(oc get route argocd-server -o jsonpath='{.spec.host}' -n openshift-gitops) \
  --username admin \
  --password $(oc extract secret/argocd-cluster --to=- --keys=admin.password -n openshift-gitops 2>&1 | tail -1)
```

- Na **linha 2** obtemos o endereço do ArgoCD
- `-n openshift-gitops` namespace onde roda o ArgoCD
- `secret/argocd-cluster` secret que contém o password do usuário `admin`

## Alguns comandos

```bash
## Listar os clusters
argocd cluster list

## Listar os SSH known host entries
argocd cert list --cert-type ssh

## Listar os repositórios
argocd repo list
```

## Projeto no ArgoCD

Por padrão o ArgoCD vem com um projeto chamado `default` e todas as aplicações - também por padrão - são criadas nele. O projeto é um recurso dentro do cluster e pode ser visto assim `oc get AppProject default -o yaml -n openshift-gitops` ou pela interface web do ArgoCD.

Vamos criar um outro projeto específico para a nossa aplicação (repositório git). Certamente um mesmo projeto no ArgoCD pode gerenciar múltiplas aplicações (repositórios git), ex: dois repositórios git que guardam front e back, que são duas aplicações distintas, podem ser gerenciadas dentro do mesmo projeto no ArgoCD.

### Criar projeto

```bash
argocd proj create jarvis \
  --dest https://kubernetes.default.svc,jarvis \
  --src git@git.myrepo.com.br:jarvishq/jarvis-api.git \
  --description 'Jarvis DEV'
```

- `-d, --dest stringArray`: endereço do cluster e namespace que esse projeto irá gerenciar recursos (ex https://192.168.99.100:8443,default)
- `-s, --src stringArray`: endereço do repositório git de onde esse projeto poderá baixar código (vai ser o endereço git da aplicação)
- Para múltiplos sources (repositórios), adicione `--src` para cada um deles no comando acima

### Recurso do projeto dentro do cluster

O recurso dentro do cluster referente a esse projeto pode ser visto assim

```bash
oc get AppProject jarvis -o yaml -n openshift-gitops
```

### Diferenças em relação ao projeto `default`

É interessante comparar com o projeto `default` para se ter noção das diferenças

```bash
icdiff -N -H \
  <(oc get AppProject default -o yaml) \
  <(oc get AppProject jarvis -o yaml)
```

> - Você pode trocar `icdiff -N -H` por `diff --side-by-side --color`
> - icdiff: <https://github.com/jeffkaufman/icdiff>

## Adicionar as chaves públicas SSH do GitLab

Para que o ArgoCD possa acessar o GitLab sem reclamar da validade da chave (https).

```bash
ssh-keyscan git.myrepo.com.br | argocd cert add-ssh --batch
```

## Adicionar um repositório do GitLab

Para que o ArgoCD possa clonar o repositório git.

É preciso criar uma chave pública-privada (ex: `ssh-keygen -t rsa -b 4096 -C "USER@argocd-REPONAME" -f /tmp/mykey/id_rsa`) para um usuário que tenha acesso ao repositório git. A chave pública deve ser adiciona ao usuário nas configurações do GitLab. A chave privada deve ser adicionada na configuração do repositório git no ArgoCD e isso será feito no comando abaixo.

```bash
argocd repo add \
  git@git.myrepo.com.br:jarvishq/jarvis-api.git \
  --ssh-private-key-path /tmp/mykey/id_rsa \
  --project jarvis
```

Checar se o ArgoCD conseguiu se conectar ao repositório git adicionado

```bash
argocd repo list
```

## Adicionar permissão para o ArgoCD maniupular recursos no namespace

O ArgoCD está no namespace `openshift-gitops` e vai controlar aplicações que serão deployadas no namespace `jarvis`. É preciso liberar permissão para que o ArgoCD possa manipular os recursos no namespace `jarvis`.

```bash
oc adm policy \
  add-role-to-user \
  admin \
  system:serviceaccount:openshift-gitops:argocd-argocd-application-controller \
  -n jarvis
```

## Isso é tudo!

Ao criar uma aplicação no ArgoCD (que roda no namespace `openshift-gitops`) ela será deployada no namespace `jarvis`.

## Referências

- <https://argo-cd.readthedocs.io/en/stable/getting_started>
- <https://argo-cd.readthedocs.io/en/stable/user-guide/projects>
- <https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories>
- <https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup>
