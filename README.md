# Um blog sobre Programação, configuração, script, Linux e afins

<https://mhagnumdw.github.io/>

## Executando local

```bash
export JEKYLL_VERSION=3.8

# entrar na pasta do projeto e executar
docker run \
  --name jekyll \
  -it \
  --rm \
  --env JEKYLL_UID=$(id -u) \
  --volume="$PWD:/srv/jekyll"  \
  --volume="jekyll_$JEKYLL_VERSION:/usr/local/bundle" \
  -p 4000:4000 \
  jekyll/jekyll:$JEKYLL_VERSION \
  jekyll serve
```

<https://github.com/envygeeks/jekyll-docker>
