# Informações técnicas sobre o projeto

Tema utilizado: <https://github.com/khusika/FeelIt>

O tema é instalado como submodule do git:

```bash
git submodule add https://github.com/khusika/FeelIt.git themes/FeelIt
```

Para verificar o status do submodule:

```bash
git submodule status
# ou apenas:
git submodule
# ou:
cat .gitmodules
```

Para atualizar apenas o tema:

```bash
git submodule update --init --recursive
```

Para incluir os submodules no pull:

```bash
git pull --recurse-submodules
```

Existe personalização do CSS do tema em `assets/css`.

Se algumas partes do tema forem sobrescritas, elas ficam na pasta `layouts/_partials`. Essa sobrescrita é um padrão do Hugo. Quando o tema for atualizado é importante fazer um diff com a versão original dos arquivos e, se houver diferença extra, o conteúdo da pasta `layouts/_partials` deve ser atualizado.
