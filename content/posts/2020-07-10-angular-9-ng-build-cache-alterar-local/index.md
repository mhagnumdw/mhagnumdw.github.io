---
author-id: mhagnumdw
categories:
- angular
date: "2020-07-10T17:06:00Z"
feature-img: assets/img/posts/angular-9-ng-build-cache-alterar-local/angular-9-ng-build-cache-alterar-local-banner.png
image: assets/img/posts/angular-9-ng-build-cache-alterar-local/angular-9-ng-build-cache-alterar-local-banner.png
tags:
- angular
- build
- performance
thumbnail: assets/img/posts/angular-9-ng-build-cache-alterar-local/angular-9-ng-build-cache-alterar-local-banner.png
title: 'Angular 9+: alterar local do cache do ng build'
---

A partir do Angular 9+ existe um cache do build que por padrão é criado dentro da pasta `./node_modules/.cache`. Vamos ver como alterar o local desse cache e também a diferença que ele faz no tempo de build.

<!--more-->

Porque alterar o local do cache? Em algumas ocasiões, por exemplo quando se trabalha com pipelines em uma ferramenta de CI/CD, pode ser necessário ter essa flexibilidade.

## Alterando o local do cache

```shell
NG_BUILD_CACHE=/novo/local/do/cache ng build
```

E é só isso. Basta dar uma olhada no conteúdo da pasta.

> 📋 No momento praticamente não há referência na internet ao buscar pelo termo `NG_BUILD_CACHE`.

## Tempo de build

<video muted autoplay controls style="width=:100%;padding: unset;">
    <source src="{{ site.baseurl }}/assets/img/posts/angular-9-ng-build-cache-alterar-local/angular-9-ng-build-cache-alterar-local.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

> 📋 No vídeo o local do cache está o padrão, como visto mais acima, dentro de `./node_modules/.cache`.

## Referências

- <https://github.com/angular/angular-cli/pull/15900>
