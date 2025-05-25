---
author-id: mhagnumdw
categories:
- Linux
date: "2017-01-26T12:57:29Z"
feature-img: assets/img/posts/fedorarhel-instalar-microsoft-core-fonts/fedora_redhat_microsoft_core_fonts.jpg
image: assets/img/posts/fedorarhel-instalar-microsoft-core-fonts/fedora_redhat_microsoft_core_fonts.jpg
tags:
- Fedora
- fonts
- RHEL
thumbnail: assets/img/posts/fedorarhel-instalar-microsoft-core-fonts/fedora_redhat_microsoft_core_fonts.jpg
title: Fedora/RHEL - Instalar Microsoft core fonts
---

Instalar as fontes da Microsoft no linux Fefora/RHEL. Para outras distros os passos são os mesmos ou bem parecidos.

<!--more-->

```shell
dnf install rpm-build cabextract ttmkfdir
```

> Para o RHEL o **cabextract** está no EPEL
>
> Para o Fedora é bom que tenha os repositórios do RPM Fusion

```shell
cd /tmp
wget http://corefonts.sourceforge.net/msttcorefonts-2.5-1.spec
```

Precisamos ajustar um endereço no arquivo acima. Na linha 62 deve haver:

```text
mirror="http://${m}.dl.sourceforge.net/project/corefonts/the%20fonts/final/"
```

Que deve ser substituída por:

```text
mirror="http://sourceforge.net/projects/corefonts/files/the%20fonts/final/"
```

Executar

```shell
rpmbuild -bb msttcorefonts-2.5-1.spec
cp ~/rpmbuild/RPMS/noarch/msttcorefonts-2.5-1.noarch.rpm /tmp
dnf install msttcorefonts-2.5-1.noarch.rpm
fc-cache /usr/share/fonts
```

Não deve ser necessário reiniciar.

Pronto!

Referências

- [https://sayaksarkar.wordpress.com/2013/06/02/installing-microsoft-truetype-fonts-in-fedora18/](https://sayaksarkar.wordpress.com/2013/06/02/installing-microsoft-truetype-fonts-in-fedora18/)
- [https://access.redhat.com/solutions/124613](https://access.redhat.com/solutions/124613)
