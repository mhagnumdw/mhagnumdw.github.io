# Um blog sobre Programação, configuração, script, Linux e afins

<https://mhagnumdw.github.io/>

## Executando local

Entrar na pasta do projeto e executar os comandos abaixo.

```bash
export JEKYLL_VERSION=4

docker run \
  --name jekyll \
  -it \
  --rm \
  --env JEKYLL_UID=$(id -u) \
  --volume="$PWD:/srv/jekyll"  \
  --volume="jekyll_$JEKYLL_VERSION:/usr/local/bundle" \
  -p 4000:4000 \
  -p 4001:4001 \
  jekyll/jekyll:$JEKYLL_VERSION \
  jekyll serve --livereload --livereload-port 4001
```

<https://github.com/envygeeks/jekyll-docker>
