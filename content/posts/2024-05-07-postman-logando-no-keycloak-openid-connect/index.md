---
author-id: mhagnumdw
categories:
- Postman
- Keycloak
date: "2024-05-07T15:03:00Z"
feature-img: assets/img/posts/postman-logando-no-keycloak-openid-connect/banner.png
image: assets/img/posts/postman-logando-no-keycloak-openid-connect/banner.png
tags:
- Postman
- Keycloak
thumbnail: assets/img/posts/postman-logando-no-keycloak-openid-connect/banner.png
title: Postman logando no Keycloak com OpenID Connect + OAuth2
---

Configurar o Postman para realizar requests para um API que precisa de autenticação no Keycloak. Será utilizado o padrão de mercado OpenID Connect com OAuth2. O fluxo do OAuth2 será o Authorization Code + PKCE.

<!--more-->

Índice

- [Criar uma coleção](#criar-uma-coleção)
- [Configurar a autorização na coleção](#configurar-a-autorização-na-coleção)
  - [Definir algumas variáveis](#definir-algumas-variáveis)
  - [Selecionar OAuth 2.0](#selecionar-oauth-20)
  - [Confirmar o WARN se estiver de acordo](#confirmar-o-warn-se-estiver-de-acordo)
  - [Configurar um novo token](#configurar-um-novo-token)
  - [Gerar um novo token para um dado usuário](#gerar-um-novo-token-para-um-dado-usuário)
  - [Gerar um novo token para outro usuário](#gerar-um-novo-token-para-outro-usuário)
- [Realizar uma requisição HTTP com o Access Token](#realizar-uma-requisição-http-com-o-access-token)
- [Alternar entre usuários](#alternar-entre-usuários)

## Criar uma coleção

Você pode usar uma coelação já existente, mas se não sabe bem o que está fazendo, crie uma nova.

![989292485d92dd5872405b6a2171cc12.png](989292485d92dd5872405b6a2171cc12.png)

Nomear a coleção:

![9b6df2af99981bf9eec9db33ea1678b1.png](9b6df2af99981bf9eec9db33ea1678b1.png)

## Configurar a autorização na coleção

A vantagem de configurar a autorização na coleção e não diretamente em um request, é que fica possível e super fácil reusar a configuração de autenticação entre todos os requests da coleção.

### Definir algumas variáveis

1. ID do cliente OAuth2 (se não souber, obtenha com o seu provedor de autenticação)
	- Será utilizado pelo Postman para obter o token
2. O endereço base para o endpoint da API
	- Será utilizado para fazer os requests para a API

![f0453454b002f6a0e8376dd8b3c869ab.png](f0453454b002f6a0e8376dd8b3c869ab.png)

### Selecionar OAuth 2.0

![21d2adb9d25d4b681cbd3310cb9ce1a3.png](21d2adb9d25d4b681cbd3310cb9ce1a3.png)

### Confirmar o WARN se estiver de acordo

![c6d3a9036c5f05fc9f1f041fefb5e5b2.png](c6d3a9036c5f05fc9f1f041fefb5e5b2.png)

### Configurar um novo token

![01700095f243bec063090e7899a553d2.png](01700095f243bec063090e7899a553d2.png)

Sobre alguns itens:

- 1. Um nome arbitrário (foi usado o nome da aplicação)
- 3. Em `Callback URL` usar o valor <https://www.postman.com>
- 4. Em `Auth URL` usar <https://app-main-sefin-keycloak-idp2.app.dev.xxxxxxxxxx.br/realms/jarvis/protocol/openid-connect/auth> (ajuste conforme seu caso)
- 5. Em `Access Token URL` usar <https://app-main-sefin-keycloak-idp2.app.dev.xxxxxxxxxx.br/realms/jarvis/protocol/openid-connect/token> (ajuste conforme seu caso)
- 8. Em `Scope` usar `openid offline_access`

### Gerar um novo token para um dado usuário

Ao final, ainda na aba `Authorization`, clicar em ambos os botões:

![939d5793e75b030e4bf9805b1709868d.png](939d5793e75b030e4bf9805b1709868d.png)

Então deve abrir uma modal...

![90ee542dc6a31bc02de661bd4ec9f3b6.png](90ee542dc6a31bc02de661bd4ec9f3b6.png)

...e também a janela de autenticação do Keycloak, solicitando usuário e senha:

![9b11c9f6994955585cc678adaaa0b12b.png](9b11c9f6994955585cc678adaaa0b12b.png)

Basta colocar as credenciais e se autenticar. Após, deve aparecer o modal abaixo ...

![ee37ddaf135afb6ba2f6e2a0c97d24eb.png](ee37ddaf135afb6ba2f6e2a0c97d24eb.png)

... e logo em seguida os dados do token:

![34fdc6ec6c71a667156372922705cc4e.png](34fdc6ec6c71a667156372922705cc4e.png)

Usando a edição apontada pela seta vermelha, **eu adicionei, ao final do nome do token, o nome do usuário para qual o token foi gerado. Isso ajuda a identificar de quem é o token,** pois você pode ter tokens para usuários diferentes durantes suas requisições.

Clicar em `Use Token`:

![5dd399295e190adcddb8f175448780c0.png](5dd399295e190adcddb8f175448780c0.png)

Então a área `Current Token` é preenhida automaticamente:

![14141548bd4b49db2d681a4afebafee9.png](14141548bd4b49db2d681a4afebafee9.png)

Observar:
- no item 1 o nome do token, que acaba identificando o usuário
- o token está no item 3
- o auto-refresh do token foi marcado no item 5
- é possível forçar um refresh manual no link `Refresh` (entre o item 3 e 4)

### Gerar um novo token para outro usuário

Basta seguir basicamente o que já foi feito. Clicar em `Clear cookies` e em seguida em `Get New Access Token`:

![939d5793e75b030e4bf9805b1709868d.png](939d5793e75b030e4bf9805b1709868d.png)

Se logar na tela do Keycloak que vai abrir. E após logar definir o nome do token e decidir se deseja usá-lo ou não:

![028f3c20899a5b9169bfe7f2521a829e.png](028f3c20899a5b9169bfe7f2521a829e.png)

## Realizar uma requisição HTTP com o Access Token

Basta criar uma requisição, na guia `Auth` selecionar o `Inherit auth from parent` e realizar a requisição

![f4e7ac5b5a50c54e865366adad4676c4.png](f4e7ac5b5a50c54e865366adad4676c4.png)

A requisição deu sucesso e é possível ver que o token foi enviado no request

![0226770549ec66cae53768dc7bbebe90.png](0226770549ec66cae53768dc7bbebe90.png)

## Alternar entre usuários

Na configuração da coleção, na aba `Authorization`, mudar de usuário conforme a imagem:

![8dcf56905cb21fbaa63426ed58e959f5.png](8dcf56905cb21fbaa63426ed58e959f5.png)
