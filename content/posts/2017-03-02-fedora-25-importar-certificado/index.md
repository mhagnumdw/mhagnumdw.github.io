---
author-id: mhagnumdw
categories:
- Linux
date: "2017-03-02T15:49:32Z"
feature-img: assets/img/posts/fedora-25-importar-certificado/nss_importar_certificado.png
image: assets/img/posts/fedora-25-importar-certificado/nss_importar_certificado.png
tags:
- Certificado
- Fedora
thumbnail: assets/img/posts/fedora-25-importar-certificado/nss_importar_certificado.png
title: Fedora 25 - Importar certificado
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
