---
layout: post
title: Fedora 25 - Importar certificado
date: 2017-03-02 15:49:32 -03:00
categories:
- Linux
tags:
- Certificado
- Fedora
author-id: mhagnumdw
image: "assets/img/posts/fedora-25-importar-certificado/nss_importar_certificado.png"
feature-img: "assets/img/posts/fedora-25-importar-certificado/nss_importar_certificado.png"
thumbnail: "assets/img/posts/fedora-25-importar-certificado/nss_importar_certificado.png"
---

Importar certificado para que seja reconhecido pelos browsers e outros softwares.

<!--more-->

Com a importação do certificado deixa-se de receber o alerta que os browsers exibem sobre conexão não segura.

#### Listar os certificados atuais

```shell
certutil -d sql:$HOME/.pki/nssdb -L
```

#### Importar um certificado

```shell
certutil -d sql:$HOME/.pki/nssdb -A -t "TCu,Cu,Tuw" -n 'MhagnumDw-CA' -i ~/Downloads/MhagnumDw-CA.cer
```

Referências

- [https://wiki.archlinux.org/index.php/Network_Security_Services](https://wiki.archlinux.org/index.php/Network_Security_Services)
