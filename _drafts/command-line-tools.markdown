# Lista refinada de utilitários de linha de comando

## Diff

### icdiff

```bash
icdiff <(oc get route app-jbossws-ws -o yaml) <(oc get route app-jbossws -o yaml)
```

https://github.com/jeffkaufman/icdiff


### vimdiff

```bash
vimdiff <(oc get route app-jbossws-ws -o yaml) <(oc get route app-jbossws -o yaml)
```


## Docker

### dive

Analisar imagens docker.

```bash
dive IMAGE:TAG

CI=true dive IMAGE:TAG
```

https://github.com/wagoodman/dive


## Análise de disco

[Linux] ncdu no lugar do du, bem melhor.

## Busca

[Linux] fd no lugar do find (chamado de fdfind): https://github.com/sharkdp/fd

[Linux] rg no lugar do grep (chamado de ripgrep): https://github.com/BurntSushi/ripgrep

## Visualizador de arquivos texto/logs

### lnav

https://github.com/tstack/lnav


## top style (monitoramento)

### bpytop

https://github.com/aristocratos/bpytop
