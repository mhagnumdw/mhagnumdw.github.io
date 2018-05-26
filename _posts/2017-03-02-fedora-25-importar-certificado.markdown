---
layout: post
title: Fedora 25 - Importar certificado
date: 2017-03-02 15:49:32 -03:00
categories:
- Linux
tags:
- Certificado
- Fedora
#author:
#  login: mhagnumdw
#  email: mhagnumdw@gmail.com
#  display_name: mhagnumdw
#  first_name: ''
#  last_name: ''
feature-img: "assets/nss_importar_certificado.png"
thumbnail: "assets/nss_importar_certificado.png"
---

Importar certificado para que seja reconhecido pelos browsers e outros softwares.

Com a importação do certificado deixa-se de receber o alerta que os browsers exibem sobre conexão não segura.

#### Listar os certificados atuais

{% highlight shell %}
certutil -d sql:$HOME/.pki/nssdb -L
{% endhighlight %}

#### Importar um certificado

{% highlight shell %}
certutil -d sql:$HOME/.pki/nssdb -A -t "TCu,Cu,Tuw" -n 'MhagnumDw-CA' -i ~/Downloads/MhagnumDw-CA.cer
{% endhighlight %}

Referências
- [https://wiki.archlinux.org/index.php/Network_Security_Services](https://wiki.archlinux.org/index.php/Network_Security_Services)