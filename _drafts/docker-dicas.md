# Setar o timezone
docker run -e TZ=America/Fortaleza -d --name sonarqube -p 9001:9000 sonarqube:8.2-community
Lista de timezones: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones

# Apagar todas as tags de uma imagem de um registry remoto
```shell
skopeo list-tags docker://registry.meudominio.com/app | \
   jq -r '.Tags' | \
   grep -P -o '".*?"' | \
   grep -P -o '[^"]+' | \
   xargs -I{} skopeo delete docker://docker://registry.meudominio.com/app:{}
```

# Apagar as imagens a partir de X dias

```shell
 - days="15" # Imagens com mais de X dias serão apagadas
 # Modo 1
 - docker image prune --all --force --filter=until=$((days * 24))h --filter=label=${IMAGE_TEMP_NAME}
 - timeago=$(date --date "$days days ago" +'%s')
 # Modo 2
 # Deleta a imagem pelo <image:tag> ao invés do ID, evita erros se a imagem estiver sendo referenciada
 - docker image ls --filter=reference="$IMAGE_TEMP_NAME" --format "{{.Repository}}:{{.Tag}};{{.CreatedAt}}" |
     while IFS=";" read -r IMAGE CreatedAt; do
       CreatedAt=$(echo "$CreatedAt" | rev | cut -c5- | rev | xargs -I{} date -d {} +%s);
       if [ "$CreatedAt" -lt "$timeago" ]; then echo "$IMAGE"; docker image rm --force "$IMAGE"; fi;
     done
```
