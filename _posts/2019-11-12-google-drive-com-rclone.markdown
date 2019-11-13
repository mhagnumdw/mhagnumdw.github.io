---
layout: post
title: 'Google Drive sincronizando com rclone'
date: 2019-11-12 22:38:00 -03:00
categories:
- Sync
tags:
- google drive
- sync
- cloud
- linux
author-id: mhagnumdw
image: "assets/img/posts/google-drive-com-rclone/google-drive-com-rclone-banner.png"
feature-img: "assets/img/posts/google-drive-com-rclone/google-drive-com-rclone-banner.png"
thumbnail: "assets/img/posts/google-drive-com-rclone/google-drive-com-rclone-banner.png"
---

Direto ao ponto: sincronizando no Linux com Google Drive e [rclone](https://rclone.org/).

<!--more-->

### Instalar e Configurar o rclone

#### Instalar
```bash
curl https://rclone.org/install.sh | sudo bash
# checar a versão
rclone version
```

#### Configurar

```bash
# executar
rclone config
```

Responder de acordo com o que está abaixo
```
n/s/q> n
name> GoogleDrive (dê o nome que preferir)
Storage> 13
client_id> (deixar em branco, ENTER)
client_secret> (deixar em branco, ENTER)
scope> 1
root_folder_id> (deixar em branco, ENTER)
service_account_file> (deixar em branco, ENTER)
Edit advanced config? (y/n) n
Use auto config? y
```

Em seguida deve aparecer um link similar ao link abaixo e o browser deve abrir automaticamente, caso não clique no link para abrir.

- http://127.0.0.1:53682/auth?state=xxxxxxxxxxxxxxxxxxxxxx

Então:

- Esse link redireciona para a tela de login com o Google.
- Confirme o login.
- Deve aparecer no browser a mensagem abaixo:

```
Success!
All done. Please go back to rclone.
```

Volte para o console (que deve ter tido alteração!) E continue respondendo:

```
Configure this as a team drive? n
y) Yes this is OK [ y ]
q) Quit config [ q ]
```

Nesse ponto toda a configuração deve está ok.

### Testando

```bash
# Lista os drivers remotos
rclone listremotes

# Listar o conteúdo
rclone ls --max-depth 1 GoogleDrive:/
# ou
rclone ls GoogleDrive:/
```

> `--max-depth 1` vai listar apenas a pasta raiz


### Especificar o que deve ser sincronizando

Nem sempre desejamos sincronizar todo o drive. Vamos especificar o que sincronizar. Para isso criar o arquivo `filter-file-GoogleDrive.txt` (dê o nome que preferir) e especificar o que deve ou não ser sincronizado, conforme conteúdo de exemplo abaixo.
- Com sinal de `+` deve sincronizar
- Com sinal de `-` não deve sincronizar
- Mais detalhes [aqui](https://rclone.org/filtering/)


```
# Sincronizar
+ /Aplicativos/**
+ /Apostilas/**
+ /Backup/**
+ /Pasta\ Com\ Espaco/**
+ /Documentos/**
+ /Gifs/**
# Nao sincronizar
- *
```


### Sincronizar do Google Drive para uma pasta local

> Adicionar a opção `--dry-run` após o `rclone sync` para ver o resultado sem efetivar as mudanças

```bash
rclone sync \
  --create-empty-src-dirs \
  --progress \
  --drive-acknowledge-abuse \
  --filter-from filter-file-GoogleDrive.txt \
  GoogleDrive:/ /root/GoogleDrive/
```

### Sincronizar de uma pasta local para o Google Drive

> Adicionar a opção `--dry-run` após o `rclone sync` para ver o resultado sem efetivar as mudanças

```bash
rclone sync \
  --create-empty-src-dirs \
  --progress \
  --drive-acknowledge-abuse \
  --filter-from filter-file-GoogleDrive.txt \
  /root/GoogleDrive/ GoogleDrive:/
```

### Sincronizando em ambos os sentidos

O `rclone sync` só faz sincronização em um único sentido. Ele só altera a pasta de destino, como pode ser visto na documentação [aqui](https://rclone.org/commands/rclone_sync/).

**Vamos usar o `rclonesync.py` para isso**, que é indicado no próprio site do `rclone` [aqui](https://github.com/rclone/rclone/wiki/Third-Party-Integrations-with-rclone#rclonesync-v2).


```bash
# download
wget https://raw.githubusercontent.com/cjnaz/rclonesync-V2/f2316cfeebda1532c343890668ec972bf3bb276e/rclonesync.py
# tornar executável
chmod +x rclonesync.py
```

#### Primeira Execução
> Na primeira sincronização é necessário executar com o parâmetro `--first-sync`

> É possível usar o parâmetro `--dry-run` para ver o resultado sem efetivar as mudanças

> Observar que primeiro a pasta local e em seguida a pasta remota (em tese mudar a ordem não deve alterar o resultado)

```bash
./rclonesync.py /root/GoogleDrive/ GoogleDrive:/ \
  --filters-file filter-file-GoogleDrive.txt \
  --verbose \
  --first-sync \
  --rclone-args --drive-export-formats=.link.html --drive-acknowledge-abuse
```

#### Execuções subsequentes
> As execuções subsequentes não devem ter o parâmetro `--first-sync`

```bash
./rclonesync.py /root/GoogleDrive/ GoogleDrive:/ \
  --filters-file filter-file-GoogleDrive.txt \
  --verbose \
  --rclone-args --drive-acknowledge-abuse --drive-skip-gdocs

# ou

./rclonesync.py /root/GoogleDrive/ GoogleDrive:/ \
  --filters-file filter-file-GoogleDrive.txt \
  --verbose \
  --rclone-args --drive-acknowledge-abuse --drive-export-formats=.link.html
```

> A opção `--drive-skip-gdocs` não sincroniza os arquivos do Google Docs, que no sistema de arquivos local seriam apenas atalhos.

> Caso queira sincronizar os arquivos do Google Docs, use a opção `--drive-export-formats=.link.html` que atualmente é documentada [aqui](https://github.com/rclone/rclone/pull/2479) e [aqui](https://rclone.org/drive/#import-export-of-google-documents)

> A opção `--drive-acknowledge-abuse` permite sincronizar arquivos que o Google Drive acha suspeitos (ex: malware).

### Automizando!

Afinal não queremos ficar executando manualmente. Vamos agendar a sincronização no cron.

```bash
crontab -e
```

Adicionar o código abaixo que agenda a execução a cada 5 minutos.

```
*/5 * * * * ./rclonesync.py /root/GoogleDrive/ GoogleDrive:/ \
  --filters-file filter-file-GoogleDrive.txt \
  --verbose \
  --rclone-args \
  --drive-acknowledge-abuse \
  --drive-export-formats=.link.html 2>&1 >> ~/GoogleDrive_rclonesync.log
```

### Referências

- https://github.com/rclone/rclone
- https://forum.rclone.org/t/bi-directional-rclone-solution/3478
- https://rclone.org/drive/
- https://github.com/cjnaz/rclonesync-V2
