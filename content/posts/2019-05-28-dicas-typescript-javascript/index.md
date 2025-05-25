---
title: 'Typescript/ECMAScript/Javascript: dicas'
date: "2019-05-28T17:52:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
resources:
- name: "featured-image"
  src: "dicas-typescript-javascript-banner.png"
categories: ["javascript"]
tags: ["typescript", "ECMAScript", "javascript"]
---

Dicas de Typescript/ECMAScript/Javascript (snippets)

<!--more-->

## Remover elementos de um Indexable Type

```typescript
const themes: {[ theme: string ]: string} = {
  'default-theme': 'Padrão',
  'dark-theme': 'Escuro'
};

// remover o elemento 'default-theme'
delete themes['default-theme'];
```

## Remover um elemento sem mudar o original

```typescript
const allThemesExceptDefault = { ...themes };
// remover o elemento 'default-theme'
delete allThemesExceptDefault['default-theme'];
```

## Juntar dois ou mais arrays usando Array Spread (ES6)

```typescript
const arr1 = [1,2,3]
const arr2 = [4,5,6]
const arr3 = [...arr1, ...arr2] //arr3 ==> [1,2,3,4,5,6]
```
