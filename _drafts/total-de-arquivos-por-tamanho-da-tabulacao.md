## Quantidade de arquivos por tamanho da tabulação (apenas os arquivos: css scss ts js html):

```shell
time find \
  -type f \( -iname \*.css -o -iname \*.scss -o -iname \*.ts -o -iname \*.js -o -iname \*.html \) \
  -not -path "./node_modules/*" \
  -not -path "./dist/*" \
  -not -path "./.git/*" | \
  while read -r file; \
    do \
      detect-indent $file | wc -c; \
    done | \
  sort -n | \
  uniq -c | \
  sort -n
```

## Quantidade de arquivos por tamanho da tabulação (todos os arquivos):

```shell
time find \
  -type f \
  -not -path "./node_modules/*" \
  -not -path "./dist/*" \
  -not -path "./.git/*" | \
  while read -r file; \
    do \
      detect-indent $file | wc -c; \
    done | \
    sort -n | \
    uniq -c | \
    sort -n
  ```
