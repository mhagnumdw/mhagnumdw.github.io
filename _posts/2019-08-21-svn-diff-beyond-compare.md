---
layout: post
title:  "SVN Diff com Beyond Compare"
date: 2019-08-21 08:47:00 -03:00
tags:
  - svn
  - diff
author-id: mhagnumdw
image: "assets/img/posts/svn-diff-beyond-compare/svn-beyond-compare-banner.png"
feature-img: "assets/img/posts/svn-diff-beyond-compare/svn-beyond-compare-banner.png"
thumbnail: "assets/img/posts/svn-diff-beyond-compare/svn-beyond-compare-banner.png"
---

Usar o Beyond Compare como ferramenta de **diff** para o SVN.

<!--more-->

> Ambiente: Linux

Criar o script `bcdiff.sh` com o conteúdo abaixo.

```shell
#!/bin/bash
/usr/bin/bcompare "$6" "$7" -title1="$3" -title2="$5" -readonly
exit 0
```

Executar
```shell
chmod +x bcdiff.sh
```

Realizar o diff
```shell
svn diff \
  --diff-cmd=bcdiff.sh \
  http://meusvn.com/minhaapp/trunk/app \
  http://meusvn.com/minhaapp/branches/app-myfeature1
```

> Para efetivar as mudanças, deixando de ser necessário especificar o parâmetro `--diff-cmd`, modificar o arquivo `$HOME/.subversion/config`. Dentro da seção `[helpers]` adicionar `diff-cmd = /my/path/bcdiff.sh`

> Para Windows ver o link nas referências

## Referências
- http://www.scootersoftware.com/support.php?zz=kb_vcs.php#svn
