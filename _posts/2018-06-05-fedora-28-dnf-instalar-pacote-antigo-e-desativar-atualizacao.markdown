---
layout: post
title: 'Fedora 28 + DNF: instalar pacote antigo e desativar atualização'
date: 2018-06-05 14:39:00 -03:00
categories:
- Linux
tags:
- Fedora
- dnf
- yum
author: mhagnumdw
feature-img: "assets/fedora-dnf-package-old-disable.png"
thumbnail: "assets/fedora-dnf-package-old-disable.png"
---

Instalar uma versão específica de um pacote e desativar sua atualização ao executar `dnf update`.

<!--more-->

Para o exemplo abaixo será utilizado o pacote do `subversion`.

#### Listar as versões do pacote
{% highlight shell %}
dnf --showduplicates list subversion
{% endhighlight %}

#### Instalar a versão desejada
{% highlight shell %}
sudo dnf install subversion-1.9.7-6.fc28
{% endhighlight %}
_Especificar a versão desejada concatenado o nome do pacote com a versão_

#### Excluir o pacote da atualização do DNF
Editar o arquivo `sudo vim /etc/dnf/dnf.conf` e adicionar a linha
```
exclude=subversion subversion-javahl subversion-lib
```

A partir de então o `dnf update` não irá atualizar os pacotes definidos acima.