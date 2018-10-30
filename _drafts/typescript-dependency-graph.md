---
layout: single
title:  "Draft Post"
header:
  teaser: "unsplash-gallery-image-2-th.jpg"
categories: 
  - Jekyll
tags:
  - edge case
---

Gerar gráfico das dependências internas

![alt text](tslint-without-node_modules.png "Dependency Graph")


$ sudo npm install --global dependency-cruiser
$ sudo dnf install graphviz
$ cd ./jarvis/frontend/ngapp
$ dependency-cruise -T dot -x '(node_modules|app.module.ts)' ./src/app/  | dot -T png > dependency-graph.png

Referências:
https://github.com/sverweij/dependency-cruiser
https://github.com/sverweij/dependency-cruiser/blob/develop/doc/real-world-samples.md
