---
layout: post
title: 'GitLab Pipeline - Acessar uma máquina remota com chave SSH'
date: 2023-07-11 09:00:00 -03:00
categories:
- GitLab
- SSH
tags:
- GitLab
- Pipeline
- SSH
author-id: mhagnumdw
image: "assets/img/posts/gitab-pipeine-job-access-remote-ssh/banner.png"
feature-img: "assets/img/posts/gitab-pipeine-job-access-remote-ssh/banner.png"
thumbnail: "assets/img/posts/gitab-pipeine-job-access-remote-ssh/banner.png"
---

Passos para dentro de um job do pipeline do GitLab, acessar uma máquina remota via ssh.

<!--more-->

## Premissas

- O job roda em um executor do tipo docker
- A imagem de container do job tem disponível o comando `ssh-agent`
- A máquina remota tem um usuário que permite login via ssh, vamos supor que o usuário se chama `sonic`
- Um par de chave pública-privada foi criado para o usuário `sonic`, algo como: `ssh-keygen -t rsa -f /tmp/sonic/sonic_key -C 'gitlab-access-user-sonic'`
- A chave pública do usuário `sonic` foi adicionada a `/home/sonic/.ssh/authorized_keys` na máquina remota
- A chave privada será usada na configuração no GitLab

## GitLab

Nas configurações de CI/CD do projeto ou do grupo, adicione uma variável chamada `SSH_PRIVATE_KEY` do tipo `File` com o conteúdo da chave privada gerada para o usuário `sonic`.

No job, a env `SSH_PRIVATE_KEY` vai conter o path para um arquivo que vai ter o conteúdo da chave privada.

Na tag `script` do job, basta isso para acessar a máquina remota:

```yaml
- eval $(ssh-agent -s)
- (cat "$SSH_PRIVATE_KEY" && echo) | tr -d '\r' | ssh-add -
- mkdir -p ~/.ssh
- chmod 700 ~/.ssh
- ssh-keyscan host-remoto.com.br >> ~/.ssh/known_hosts
- chmod 600 ~/.ssh/known_hosts
- ssh sonic@host-remoto.com.br echo "acesso a maquina remota ok"
```

Se tiver problema de validação da chave do host, revise o comando `ssh-keyscan`. Um workaround nessa segurança é adicionar `-o StrictHostKeyChecking=no` ao comando `ssh`.

## Referências

- <https://gitlab.com/gitlab-examples/ssh-private-key/-/blob/master/.gitlab-ci.yml>
- <https://docs.gitlab.com/ee/ci/ssh_keys/>
