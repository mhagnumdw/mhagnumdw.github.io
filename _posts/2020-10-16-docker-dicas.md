---
layout: post
title: 'docker: dicas'
date: 2020-10-16 16:25:00 -03:00
categories:
- docker
tags:
- docker
- docker-dicas
author-id: mhagnumdw
image: "assets/img/posts/docker-dicas/banner.png"
feature-img: "assets/img/posts/docker-dicas/banner.png"
thumbnail: "assets/img/posts/docker-dicas/banner.png"
---

Dicas de docker.

<!--more-->

## Setar o timezone no container

```bash
docker run -e TZ=America/Fortaleza -d --name sonarqube -p 9001:9000 sonarqube:8.2-community
```

> 📋 Lista de timezones: <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones>

## Apagar todas as tags de uma imagem de um registry remoto

```bash
skopeo list-tags docker://registry.meudominio.com/app | \
  jq -r '.Tags' | \
  grep -P -o '".*?"' | \
  grep -P -o '[^"]+' | \
  xargs -I{} skopeo delete docker://registry.meudominio.com/app:{}
```

### Apagar as imagens a partir de X dias (modo 1)

```bash
days="15" # Imagens com mais de X dias serão apagadas

docker image prune --all --force \
  --filter=until=$((days * 24))h \
  --filter=label=${IMAGE_TEMP_NAME}
```

### Apagar as imagens a partir de X dias (modo 2)

```bash
days="15" # Imagens com mais de X dias serão apagadas

timeago=$(date --date "$days days ago" +'%s')

# Deleta a imagem pelo <image:tag> ao invés do ID, evita erros se a imagem estiver sendo referenciada
docker image ls --filter=reference="$IMAGE_TEMP_NAME" --format "{{.Repository}}:{{.Tag}};{{.CreatedAt}}" |
  while IFS=";" read -r IMAGE CreatedAt; do
    CreatedAt=$(echo "$CreatedAt" | rev | cut -c5- | rev | xargs -I{} date -d {} +%s);
      if [ "$CreatedAt" -lt "$timeago" ]; then echo "$IMAGE"; docker image rm --force "$IMAGE"; fi;
  done
```

### SELinux e `/var/run/docker.sock`

Ao tentar executar qualquer comando docker com SELinux ativado e der erro de acesso ao `/var/run/docker.sock`, primeiro verifique se desativando o SELinux se o erro some:

```bash
sudo setenforce 0
getenforce
```

Se o erro sumir, reative o SELinux:

```bash
sudo setenforce 1
getenforce
```

E execute:

```bash
cd /tmp
grep "docker.sock" /var/log/audit/audit.* | audit2allow -M dockersock
semodule -i dockersock.pp
```

### Exportar e importar uma imagem

```bash
# exportar
docker save jenkins:2.46.1 | gzip -9 > /tmp/jenkins-2.46.1.docker-image.gz

# importar
zcat /tmp/jenkins-2.46.1.docker-image.gz | docker load
```
