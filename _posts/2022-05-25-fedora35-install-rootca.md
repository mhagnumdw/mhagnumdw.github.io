---
layout: post
title: 'Fedora 35 - Instalando uma Autoridade Certificadora (rootCA)'
date: 2022-05-25 11:44:00 -03:00
categories:
- ferdora
tags:
- certificado
author-id: mhagnumdw
image: "assets/img/posts/fedora35-install-rootca/banner.png"
feature-img: "assets/img/posts/fedora35-install-rootca/banner.png"
thumbnail: "assets/img/posts/fedora35-install-rootca/banner.png"
---

Como instalar uma Autoridade Certificadora (rootCA). Vamos utiliza o Fedora 35.

<!--more-->

## Caso de uso

Ao acessar a api .....

## Solução

O IBGE mudou o scheme e o subdomínio de http://api.sidra.ibge.gov.br para https://apisidra.ibge.gov.br

Chrome e Firefox reconhecem o certificado do https://apisidra.ibge.gov.br

Mas o curl e Java não estão reconhecendo. A solução é instalar o rootCA da Sectigo.

Abaixo os passos que executei simulando essa instalação com sucesso:

```bash
docker run --rm -it fedora:35 bash
# falha
curl -i --location --request GET 'https://apisidra.ibge.gov.br/values/t/7063/n1/all/P/202112/v/44/h/n'

sudo dnf update ca-certificates

# falha
curl -i --location --request GET 'https://apisidra.ibge.gov.br/values/t/7063/n1/all/P/202112/v/44/h/n'

# verifiquei que o rootCA da Sectigo não está instalado
trust list | grep -i sectigo

# Baixei o certificado em /root de
# https://sectigo.com/knowledge-base/detail/Sectigo-Intermediate-Certificates/kA01N000000rfBO
# [Download] Sectigo RSA Domain Validation Secure Server CA [ Intermediate ]
# ver anexo

dnf install -y openssl

# convertendo o rootCA de crt para pem
openssl x509 -in SectigoRSADomainValidationSecureServerCA.crt -out SectigoRSADomainValidationSecureServerCA.pem -outform PEM

# instalando no sistema
cp SectigoRSADomainValidationSecureServerCA.pem /etc/pki/ca-trust/source/anchors/
update-ca-trust

# verificando que o sistema agora tem o rootCA da Sectigo
trust list | grep -i sectigo

# agora funciona com sucesso
curl -i --location --request GET 'https://apisidra.ibge.gov.br/values/t/7063/n1/all/P/202112/v/44/h/n'
```

## Referências

- <https://docs.fedoraproject.org/en-US/quick-docs/using-shared-system-certificates/>
- <https://ajmaradiaga.com/Adding-trusting-CA-Fedora/>
- <https://sectigo.com/knowledge-base/detail/Sectigo-Intermediate-Certificates/kA01N000000rfBO>
- <https://access.redhat.com/solutions/1435913>
