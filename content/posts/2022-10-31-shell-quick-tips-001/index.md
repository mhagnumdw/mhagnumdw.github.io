---
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"

categories:
- shell
date: "2022-10-31T09:50:00Z"

resources:
- name: "featured-image"
  src: "banner.png"
tags:
- find

title: 'Shell - Quick Tips #001'
---

Listar o conteúdo de uma pasta com dezenas de milhares de arquivos. Ao usar o ls é comum recebermos a mensagem: _Argument list too long_.

<!--more-->

## Solução

```bash
find ./ -maxdepth 1 -type f -name 'uc1318*'
```

- `-maxdepth 1` desce apenas um nível, no caso inspeciona apenas a pasta corrente
- `-type f` listar apenas os arquivos, não lista as pastas
- `-name 'uc1318*'` especifica que o padrão do nome do arquivo deve começar com uc1318
