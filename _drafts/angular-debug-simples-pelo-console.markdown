---
layout: post
title: 'Angular: debugando de forma simples pelo console'
date: 2020-02-25 15:12:00 -03:00
categories:
- Angular
tags:
- angular
- debug
author-id: mhagnumdw
image: "assets/img/posts/angular-debug-console/xxx.png"
feature-img: "assets/img/posts/angular-debug-console/xxx.png"
thumbnail: "assets/img/posts/angular-debug-console/xxx.png"
---

```javascript
var idInterval = setInterval(() => {
    const path = 'body > app-root > app-base > mat-sidenav-container > mat-sidenav-content > div > app-about';
    const element = document.querySelector(path);
    if (element) {
      const total = ng.probe(element).componentInstance.loadingService.total();
      console.log('total:', total);
    } else {
      console.warn('Elemento não está no DOM:', path);
    }
}, 1000);
```

```javascript
clearInterval(idInterval);
```

// No Chrome:
// ng.probe($0)
// $0 é o elemento selecionado no web console por meio do inspect
