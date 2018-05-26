---
layout: post
title: Fedora/RHEL - Instalar Microsoft core fonts
date: 2017-01-26 12:57:29 -03:00
categories:
- Linux
tags:
- Fedora
- fonts
- RHEL
#author:
#  login: mhagnumdw
#  email: mhagnumdw@gmail.com
#  display_name: mhagnumdw
#  first_name: ''
#  last_name: ''
feature-img: "assets/fedora_redhat_microsoft_core_fonts.jpg"
thumbnail: "assets/fedora_redhat_microsoft_core_fonts.jpg"
---

Instalar as fontes da Microsoft no linux Fefora/RHEL. Para outras distros os passos são os mesmos ou bem parecidos.

### Executar
{% highlight shell %}
dnf install rpm-build cabextract ttmkfdir
{% endhighlight %}

_Para o RHEL o **cabextract** está no EPEL_  
_Para o Fedora é bom que tenha os repositórios do RPM Fusion_

### Executar
{% highlight shell %}
cd /tmp
wget http://corefonts.sourceforge.net/msttcorefonts-2.5-1.spec
{% endhighlight %}

Precisamos ajustar um endereço no arquivo acima. Na linha 62 deve haver:
```
mirror="http://${m}.dl.sourceforge.net/project/corefonts/the%20fonts/final/"
```

Que deve ser substituída por:
```
mirror="http://sourceforge.net/projects/corefonts/files/the%20fonts/final/"
```

### Executar
{% highlight shell %}
rpmbuild -bb msttcorefonts-2.5-1.spec
cp ~/rpmbuild/RPMS/noarch/msttcorefonts-2.5-1.noarch.rpm /tmp
dnf install msttcorefonts-2.5-1.noarch.rpm
fc-cache /usr/share/fonts
{% endhighlight %}

Não deve ser necessário reiniciar.

Pronto!

Referências

- [https://sayaksarkar.wordpress.com/2013/06/02/installing-microsoft-truetype-fonts-in-fedora18/](https://sayaksarkar.wordpress.com/2013/06/02/installing-microsoft-truetype-fonts-in-fedora18/)
- [https://access.redhat.com/solutions/124613](https://access.redhat.com/solutions/124613)