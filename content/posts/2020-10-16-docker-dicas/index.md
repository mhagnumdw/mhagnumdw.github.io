---
title: 'docker: dicas'
date: "2020-10-16T16:25:00Z"
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"
resources:
- name: "featured-image"
  src: "banner.png"
categories: ["docker"]
tags: ["docker", "docker-dicas"]
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

Para uma máquina remota via ssh:

```bash
# exportar e importar
docker save jenkins:2.46.1 | gzip | ssh user@IP 'zcat | docker load'
```

### Alterar range padrão das redes internas

Algo que pode ser necessário é ter que mudar a faixa de rede padrão do docker por conflitar com outras redes, como uma VPN por exemplo.
Algo comum nesse sentido é alterar o `bip` dentro do `daemon.json` do docker. Porém essa estratégia falha ao criar novas interfaces, pois o _docker_ irá usar um _pool_ de endereços padrões.

Então para modificar esse _pool_ dentro de `/etc/docker/daemon.json` altere:

```json
{
    "default-address-pools": [
        {
        "base": "172.18.0.1/16",
        "size": 24
        }
    ]
}
```

Agora basta reiniciar o _docker_:

```bash
sudo service docker restart
```

## Backup de volumes (Opção 1)

```bash
# listar volumes
docker volume ls

# backup de um volume para um novo volume
VOLUME_NAME="sonarqube_logs" ; \
docker run --rm \
  -v $VOLUME_NAME:/volume_origem:ro \
  -v bkp_$VOLUME_NAME:/volume_destino \
  alpine \
  cp -ra /volume_origem /volume_destino
```

## Backup de volumes (Opção 2)

```bash
# backup para o sistema de arquivos
VOLUME_NAME="sonarqube_logs" ; \
docker run --rm \
  -v $VOLUME_NAME:/volume_origem:ro \
  -v /var/backup/volumes-sonar:/volume_destino \
  alpine \
  tar -cf /volume_destino/$VOLUME_NAME.tar -C /volume_origem .

# restore
VOLUME_NAME="sonarqube_logs" ; \
docker run --rm \
  -v $VOLUME_NAME:/volume_destino \
  -v /var/backup/volumes-sonar:/volume_origem \
  alpine \
  tar -xf /volume_origem/$VOLUME_NAME.tar -C /volume_destino
```
