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
