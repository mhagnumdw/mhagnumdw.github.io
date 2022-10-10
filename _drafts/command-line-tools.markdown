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


## Docker / Docker Manager

### dive

Analisar imagens docker.

```bash
dive IMAGE:TAG

CI=true dive IMAGE:TAG
```

https://github.com/wagoodman/dive

### lazydocker

https://github.com/jesseduffield/lazydocker


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

### gotop

https://github.com/xxxserxxx/gotop

## JVM

- https://github.com/ajermakovics/jvm-mon
- https://github.com/patric-r/jvmtop


## PS1

- https://github.com/jonmosco/kube-ps1


## git

### gitbatch

- https://github.com/isacikgoz/gitbatch

### gita

- https://github.com/nosarthur/gita

## Vídeo

### catt - transmitir para o Chromecast

- https://github.com/skorokithakis/catt

## Código

### cloc - Contar linhas de código com diversas estatísticas

- https://github.com/AlDanial/cloc


## Disk Usage

### DUF

- https://github.com/muesli/duf
