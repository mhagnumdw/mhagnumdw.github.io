---
author-id: mhagnumdw
categories:
- database
date: "2020-07-29T15:20:00Z"
feature-img: assets/img/posts/database-diff/database-diff-banner.png
image: assets/img/posts/database-diff/database-diff-banner.png
tags:
- database
- diff
thumbnail: assets/img/posts/database-diff/database-diff-banner.png
title: Database diff com SQL Workbench/J
---

Visualizar o _diff_ do esquema entre duas bases de dados - usando `jdbc`. Para o exemplo vamos utilziar o banco de dados [H2](https://www.h2database.com/) e a ferramenta [SQL Workbench/J](https://www.sql-workbench.eu/). Ao final teremos um script `SQL` para ser aplicado no banco destino para que ele fique igual ao banco de origem.

<!--more-->

Download do `SQL Workbench/J` [nessa página](https://www.sql-workbench.eu/downloads.html) ou direto por esse [link](https://www.sql-workbench.eu/Workbench-Build125-with-optional-libs.zip). Após o download extrair para uma pasta.

No Windows, executar `SQLWorkbench64.exe`. No linux `./sqlwbconsole.sh`.

## Criar profiles

Precisamos criar dois profiles das duas instâncias de banco de dados, conforme vídeo abaixo.

<video muted controls style="width=:100%;padding: unset;">
    <source src="{{ site.baseurl }}/assets/img/posts/database-diff/sql-workbench-criar-profiles.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

## Gerar o _diff_

Agora, no terminal, abrir o console `SQL Workbench/J` e executar o script que vai gerar o `SQL` de _diff_ que deve ser aplicado no destino. Ver vídeo abaixo.

<video muted controls style="width=:100%;padding: unset;">
    <source src="{{ site.baseurl }}/assets/img/posts/database-diff/sql-workbench-gerar-diff.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

> 📋 O `sql` resultante deve ser checado. Como visto no vídeo acima, pode não ser muito preciso, por exemplo: para o Oracle o drop de tabelas não foi gerado, mas para o Postgres foi.

## Referências

- <https://www.sql-workbench.eu/>
- <https://www.sql-workbench.eu/manual/compare-commands.html>
