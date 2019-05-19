---
layout: post
title: Exibir branch do Git no prompt (Linux/Cygwin)
date: 2016-09-05 11:25:10 -03:00
categories:
- Git
tags:
- Git
- Cygwin
- Linux
author: mhagnumdw
feature-img: "assets/img/posts/exibir-branch-do-git-no-prompt-linuxcygwin/exibir_branch_do_git_no_prompt.png"
thumbnail: "assets/img/posts/exibir-branch-do-git-no-prompt-linuxcygwin/exibir_branch_do_git_no_prompt.png"
---

Os passos abaixo descrevem como exibir a branch do git no prompt de comando.

<!--more-->

## Fedora/RHEL/Cygwin

{% highlight shell %}
cd ~
wget https://raw.githubusercontent.com/git/git/8976500cbbb13270398d3b3e07a17b8cc7bff43f/contrib/completion/git-prompt.sh
mv git-prompt.sh .git-prompt.sh
chmod +x .git-prompt.sh
{% endhighlight %}

## Editar .bashrc adicionando as linhas

```
#Mostrando o branch do git no prompt  
source ~/.git-prompt.sh
```

Se Fedora/RHEL, adicionar

```
export PS1='\[\e[1;36m\][\u@\h \W]\[\033[01;34m\]$(__git_ps1)\[\e[0m\]\$ '
```

Se Cygwin, adicionar

```
export PS1='\[\e]0;\w\a\]\n\[\e[32m\]\u@\h \[\e[33m\]\w\[\033[01;34m\]$(__git_ps1) \[\e[0m\]\n$ '
```

Salvar o arquivo .bashrc

## Efetivar a mudança do .bashrc

{% highlight shell %}
source .bashrc
{% endhighlight %}

A partir daqui ao entrar em um repositório git o branch será exibido no prompt.

![Exibir_branch_do_Git_no_prompt]({{ site.baseurl }}/assets/img/posts/exibir-branch-do-git-no-prompt-linuxcygwin/exibir_branch_do_git_no_prompt.png)

[Mirror para o git-prompt.sh](https://drive.google.com/open?id=0B80EagoWEV2xa3dQRllrOTQxOFE)