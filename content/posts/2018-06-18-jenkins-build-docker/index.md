---
author-id: mhagnumdw
categories:
- Jenkins
date: "2018-06-18T09:56:00Z"
feature-img: assets/img/posts/jenkins-build-docker/jenkins-build-docker-v2.png
image: assets/img/posts/jenkins-build-docker/jenkins-build-docker-v2.png
tags:
- Fedora
- Docker
- Container
- Jenkins
thumbnail: assets/img/posts/jenkins-build-docker/jenkins-build-docker-v2.png
title: Jenkins buildando dentro de container (Docker)
---

Executar o build do Jenkins dentro de um container.

<!--more-->

## Instalando Docker

```shell
# instalando
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce

# adicionando usuario ao grupo docker para evitar usar sudo
sudo groupadd docker
sudo gpasswd -a $USER docker
sudo newgrp docker ou logout/login

# ativando na inicialização do sistema e iniciando o serviço
sudo systemctl enable docker
sudo systemctl start docker

# a execução dos comando abaixo com sucesso indica que o docker foi instalado corretamente
docker images
docker run hello-world
```

## Configurando Docker

Se o acesso ao daemon do docker é feito de uma máquina remota, é necessário configurar o `hosts` no arquivo `/etc/docker/daemon.json`, em outras palavras, se o jenkins está em uma máquina e o docker em outra, essa configuração é necessária.

Adicionar o conteúdo abaixo ao arquivo `/etc/docker/daemon.json`

```json
{
  "hosts": ["tcp://0.0.0.0:2375"]
}
```

> **Essa configuração expõe o acesso ao daemon do docker a partir de todas as interfaces de rede. Para garantir a segurança tem que usar certificado: <https://docs.docker.com/engine/security/https/>**

Reiniciar o docker

```shell
sudo systemctl restart docker
```

## Configurando Jenkins

Instalar o plugin Docker Plugin: <https://wiki.jenkins.io/display/JENKINS/Docker+Plugin>

![Jenkins - Instalando plugin Docker]({{ site.baseurl }}/assets/img/posts/jenkins-build-docker/instalando-plugin-docker.png)

### Configurar plugin

1. abrir o Jenkins
1. menu `Manage Jenkins`
1. menu `Configure System`
1. na seção `Cloud`
   1. clicar em `Add a new cloud` > `Docker` > preencher os campos
   1. Name: `docker cloud`
   1. Docker Host URI: `unix:///var/run/docker.sock` ou `tcp://IP-MAQUINA-DOCKER:2375`
   1. clicar em `Test Connection` e se tudo ok deve exibir a versão do docker
   1. Enabled: `check`
   1. Container Cap: `5`
   1. clicar em `Docker Agent templates...`
   1. clicar em `Add Docker Template`
   1. Labels: `docker-slave`
   1. Enabled: `check`
   1. Docker Image: `jenkins/ssh-slave`
   1. clicar em `Container settings...`
   1. Remote Filing System Root: `/home/jenkins`
   1. Usage: `Only build jobs with label expressions matching this node`
   1. Connect method: `Connect with SSH`
   1. SSH Key: `Inject SSH Key`
   1. User: `jenkins`
   1. Pull strategy: `Pull once and update latest`
1. clicar em `Save`

### Overview da configuração

![Jenkins - Docker Conf Overview]({{ site.baseurl }}/assets/img/posts/jenkins-build-docker/plugin-docker-conf-overview.gif)

## Criar Job (ou alterar um existente)

### Configurar o build para executar em container

1. abrir as configurações do job
1. marcar a opção `Restrict where this project can be run`
1. Label Expression: `docker-slave`
![Jenkins - Restrict where this project can be run]({{ site.baseurl }}/assets/img/posts/jenkins-build-docker/restrict-where-this-project-can-be-run.png)

### Buildar

![Jenkins - Buildando dentro de container]({{ site.baseurl }}/assets/img/posts/jenkins-build-docker/buildando-dentro-de-container.gif)

## Versões/Referências

- Jenkins: 2.121.1
- Docker client/server: 18.03.1-ce
- <https://wiki.jenkins.io/display/JENKINS/Docker+Plugin>
- <https://hub.docker.com/r/jenkins/ssh-slave>
