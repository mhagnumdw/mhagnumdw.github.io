# Um blog sobre Programação, configuração, script, Linux e afins

<https://mhagnumdw.github.io/>

## Executando local

Entrar na pasta do projeto e executar os comandos abaixo.

```bash
export JEKYLL_VERSION=4.1.0

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

Adicionar a opção:

- `--future` para exibir os posts com data futura
- `--drafts` para exibir os posts na pasta `_drafts`
- `--profile` para estatíticas da compilação
- `--trace` para exibir um log detalhado
- `--help` para exibir todas as opções

<https://github.com/envygeeks/jekyll-docker>

## Comprimindo vídeos

```bash
ffmpeg -i \
  VIDEO-DE-ENTRADA.mp4 \
  -vcodec libx264 \
  -f mp4 \
  -preset slow \
  -movflags +faststart \
  VIDEO-DE-SAIDA-QUE-DEVE-SER-MENOR.mp4
```

## Minificar/comprimir js, css, imagens etc

```bash
cd assets/
npm install gulp-cli -g
npm install
gulp default
# dica: executar "git status" para ver as mudanças
git status
```

## Comprimir imagens online

<https://tinypng.com/>

## Buscar todas as categories e tags já existentes

```bash
# categories
rg --multiline --no-heading --no-filename '(?s)categories:.*?tags:' | sort | uniq -c | sort -n

# tags
rg --multiline --no-heading --no-filename '(?s)tags:.*?author-id:' | sort | uniq -c | sort -n
```
