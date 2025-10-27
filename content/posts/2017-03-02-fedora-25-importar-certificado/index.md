---
title: Fedora 25 - Importar certificado
date: "2017-03-02T15:49:32Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
avatarURL: "/images/authors/dwouglas.jpg"
resources:
- name: "featured-image"
  src: "nss_importar_certificado.png"
categories: ["Linux"]
tags: ["Certificado", "Fedora"]
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
