---
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"

date: "2019-05-28T15:28:00Z"

resources:
- name: "featured-image"
  src: "banner-tslint-without-node_modules.png"
tags:
- typescript

title: Typescript - Gráfico de dependências
---

Gerar um gráfico das dependências.

<!--more-->

```shell
# instalar dependências
sudo npm install --global dependency-cruiser
sudo dnf install graphviz
# acessar a pasta raiz do projeto
cd ./jarvis/frontend/ngapp
# gerar o gráfico
# -x: a regular expression for excluding modules
$ dependency-cruise -T dot -x '(node_modules|app.module.ts)' ./src/app/  | dot -T png > dependency-graph.png
```

## Resultado

![Gráfico das dependências]({{ "/assets/img/posts/typescript-dependency-graph/tslint-without-node_modules.png" | relative_url }})

## Referências

1. <https://github.com/sverweij/dependency-cruiser>
1. <https://github.com/sverweij/dependency-cruiser/blob/develop/doc/real-world-samples.md#typescript>
