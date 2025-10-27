---
title: 'Richfaces 3.3.4: rich:editor com spellcheck do browser'
date: "2019-05-28T16:05:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
avatarURL: "/images/authors/dwouglas.jpg"
resources:
- name: "featured-image"
  src: "richfaces_logo.jpeg"
categories: ["richfaces"]
tags: ["richfaces"]
---

Para ativar o spell check no `rich:editor`, usando os recursos do próprio browser, basta ativar a opção `gecko_spellcheck`, exemplo:

<!--more-->

```html
<rich:editor theme="advanced" value="#{mensagemBean.entidade.descricao}" required="true" viewMode="visual"
    width="600" height="300" label="Descrição" language="pt">
    <f:param name="gecko_spellcheck" value="true" />
    <f:param name="browser_spellcheck" value="true" /> <!-- sem importância para essa versão -->
    <f:param name="theme_advanced_toolbar_location" value="top" />
    <f:param name="theme_advanced_toolbar_align" value="left" />
    <f:param name="theme_advanced_buttons2" value="bullist, numlist, outdent, indent, undo, redo, spellchecker" />
    <f:param name="theme_advanced_buttons3" value="" />
</rich:editor>
```

## Resultado

![Spellcheck]({{ "/assets/img/posts/richfaces_3x_editor_ativar_spell_check/richfaces_3x_editor_ativar_spell_check.png" | relative_url }})

## Observação

- O Richfaces 3.3.4 utiliza o TinyMCE versão 3.2.5 que pode ser visto no arquivo:
richfaces-ui-3.3.4.Final.jar/org/richfaces/renderkit/html/scripts/tiny_mce/tiny_mce.js

## Referências

- <https://www.tiny.cloud/docs/configure/spelling/#gecko_spellcheck>
