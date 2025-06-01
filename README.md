# Um blog sobre Programação, configuração, script, Linux e afins

<https://mhagnumdw.github.io/>

// TODOs: ainda sem funcionar
- [ ] Tag `<video`
- [ ] A busca com lunr.js não está case-insensitive
- [ ] Disqus (sistema de comentários)

## Executando o blog localmente

Primeiro é preciso baixar o projeto:

```bash
git clone git@github.com:mhagnumdw/mhagnumdw.github.io.git
cd mhagnumdw.github.io
```

Em seguida use uma das opções abaixo para acessar o site do blog localmente.

### Com Docker (recomendado)

Entrar na pasta do projeto e executar:

```bash
docker run --rm -it \
  -v $(pwd):/src \
  -p 1313:1313 \
  hugomods/hugo:base-non-root-0.145.0 \
  server
```

Acessar <http://localhost:1313/techblog>

Para executar no modo de produção adicionar a env `--env HUGO_ENV="production"` ou o parâmetro `server --environment production`.

### Com o hugo instalado no sistema

[Instalar](https://gohugo.io/getting-started/installing) o Hugo.

Entrar na pasta do projeto e executar:

```bash
hugo server --disableFastRender --buildDrafts
```

Acessar <http://localhost:1313/techblog>

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
