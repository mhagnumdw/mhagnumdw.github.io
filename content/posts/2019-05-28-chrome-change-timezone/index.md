---
author: mhagnumdw
authorLink: "https://mhagnumdw.github.io/"

date: "2019-05-28T18:10:00Z"

resources:
- name: "featured-image"
  src: "chrome-change-timezone-logo.png"
tags:
- chrome
- timezone

title: Mudar o timezone do Chrome
---

Isso é bom pra realizar alguns testes onde servidor e cliente estão em timezones diferentes.

<!--more-->

## Por linha de comando (linux)

```shell
# mudar o TZ de acordo com a necessidade
TEMPDIR=`mktemp -d` TZ='US/Pacific' google-chrome "--user-data-dir=$TEMPDIR"
```

## Usando extensão

<https://chrome.google.com/webstore/detail/change-timezone-time-shif/nbofeaabhknfdcpoddmfckpokmncimpj>

## Sites

### Verificar se o timezone mudou

<https://whatismytimezone.com/>

### Cálculo de horário

Dado um horário + timezone e fonecendo o timezone de destino, calcula o horário do destino

<https://www.calculator.net/time-zone-calculator.html?today=11%2F01%2F2018&tctime=09%3A34%3A41&tcfrom=%2B11%3A00&tcto=-03%3A00&x=94&y=24>

## Referências

- <https://stackoverflow.com/questions/16448754/how-to-use-a-custom-time-in-browser-to-test-for-client-vs-server-time-difference>
- <https://gist.github.com/prasadsilva/225fd0394a51e52bf62f>
