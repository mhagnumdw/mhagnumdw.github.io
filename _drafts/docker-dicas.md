# Setar o timezone
docker run -e TZ=America/Fortaleza -d --name sonarqube -p 9001:9000 sonarqube:8.2-community
Lista de timezones: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
