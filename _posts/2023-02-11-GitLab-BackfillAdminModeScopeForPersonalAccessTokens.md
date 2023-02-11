---
layout: post
title: 'GitLab: BackfillAdminModeScopeForPersonalAccessTokens'
date: 2023-02-11 17:27:18 -03:00
categories:
- GitLab
tags:
- GitLab
- Update
author-id: mhagnumdw
image: "assets/img/posts/GitLab-BackfillAdminModeScopeForPersonalAccessTokens/banner.png"
feature-img: "assets/img/posts/GitLab-BackfillAdminModeScopeForPersonalAccessTokens/banner.png"
thumbnail: "assets/img/posts/GitLab-BackfillAdminModeScopeForPersonalAccessTokens/banner.png"
---

Problema com a background migration `BackfillAdminModeScopeForPersonalAccessTokens: personal_access_tokens` que falha após atualizar o GitLab da versão 15.4.6 para 15.8.1.

<!--more-->

> No momento o Google não retorna nada para `"BackfillAdminModeScopeForPersonalAccessTokens"`, mas buscando direto no GitLab.com achei uma [issue](https://gitlab.com/gitlab-org/gitlab/-/issues/388935). Curiosidamente [bing.com](https://www.bing.com/search?q=%22BackfillAdminModeScopeForPersonalAccessTokens%22) e [duckduckgo.com](https://duckduckgo.com/?q=%22BackfillAdminModeScopeForPersonalAccessTokens%22) acham.

## Problema

![git-recentb]({{ site.baseurl }}/assets/img/posts/GitLab-BackfillAdminModeScopeForPersonalAccessTokens/problema.png)

## Causa do Problema

Na máquina do GitLab, na linha de comando, entre no prompt do GitLab DB

```bash
gitlab-rails db
```

E execute

```sql
SELECT t.id,u.name,t.name,t.revoked,t.scopes
FROM personal_access_tokens t
JOIN users u ON t.user_id = u.id;
```

O problema são os escopos começando com `:`

![git-recentb]({{ site.baseurl }}/assets/img/posts/GitLab-BackfillAdminModeScopeForPersonalAccessTokens/causa-do-prolema.png)

## Soluções

### Remover os `:` dos escopos

Prefiro essa. Remover os `:` dos escopos (`replace` só remove a primeira ocorrência da string, ótimo!)

```sql
UPDATE personal_access_tokens
SET scopes = replace(scopes, ':', '');
```

Ou você pode revogar os tokens, conforme mais abaixo.

### Revogar os tokens

Revogar os tokens que possuem os `:` no escopo, algo como

```sql
UPDATE personal_access_tokens
SET revoked = 't'
WHERE id in (4, 2 ,6);
```

## Executar a background migration manualmente

![git-recentb]({{ site.baseurl }}/assets/img/posts/GitLab-BackfillAdminModeScopeForPersonalAccessTokens/executar-background-migration.png)

Que deverá ser executada com sucesso.

## Referências

- <https://gitlab.com/gitlab-org/gitlab/-/issues/388935>
