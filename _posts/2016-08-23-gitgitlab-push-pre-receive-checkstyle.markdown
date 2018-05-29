---
layout: post
title: Git/GitLab + push + pre-receive + Checkstyle
date: 2016-08-23 19:19:24 -03:00
categories:
- Git
- Java
- Linux
tags:
- Checkstyle
- GitLab
- hook
author: mhagnumdw
feature-img: "assets/test_v3.png"
thumbnail: "assets/test_v3.png"
---

Objetivo: realizar checagem de estilo de código com o checkstyle no hook pre-receive do git. Quando for realizado o push, antes de sua efetivação, a análise no código será realizada. O push será rejeitado se a análise contiver WARN's.

## Criando o hook pre-receive

1. Entrar no diretório:  
   - GitLab Instalação convencional:  
   `/home/git/repositories/<group>/<project>.git`  
   - GitLab Instalação usando o Omnibus:  
   `/var/opt/gitlab/git-data/repositories/<group>/<project>.git`
1. Criar a pasta: `custom_hooks`
1. Criar o arquivo: `pre-receive` (neste arquivo ficará o código do hook)
1. Tornar o arquivo executável
1. Escrever o código/script do hook pre-receive que pode ser em qualquer linguagem. O cabeçalho do arquivo deve indicar o tipo da linguagem. Exemplo: script em shell/bash o cabeçalho deve ser `#!/bin/bash`

## Script do hook pre-receive para o checkstyle

Abaixo um código funcional que faz a checagem de estilo de código. O push será rejeitado se a análise contiver WARN's.

{% highlight shell linenos %}
#!/bin/bash

#Ideia original de obter os arquivos das revisões e gravar em disco:
#https://gist.github.com/hartfordfive/9670011

echo
echo "### Iniciando git pre-receive para checagem de estilo de código (checkstyle) ###"
# See https://www.kernel.org/pub/software/scm/git/docs/githooks.html#pre-receive
echo

echo "PWD: "`pwd`

echo

echo "Criando pasta temporária"
TEMPDIR=`mktemp -d`
trap "echo Removendo pasta temporária se existir: ${TEMPDIR}; rm -rf ${TEMPDIR} &> /dev/null; exit" INT TERM EXIT

CHECKSTYLE_REGEX_WARN='^\[WARN\] .+?:\[0-9\]+:\[0-9\]+?: (.*)$'
CHECKSTYLE_JAR='./custom_hooks/checkstyle-7.1-all.jar'
CHECKSTYLE_XML='./custom_hooks/TechThingsCool_CheckStyle_Google.xml'
EXCLUDE_FOLDER="$TEMPDIR/src/main/metamodel" #Pasta para excluir da checagem de estilo

echo
echo "Informações:"
echo "CMD: $0"
echo "ARGUMENTS: $@"
echo "TEMPDIR: $TEMPDIR"
echo "CHECKSTYLE_REGEX_WARN: $CHECKSTYLE_REGEX_WARN"
echo "CHECKSTYLE_JAR: $CHECKSTYLE_JAR"
echo "CHECKSTYLE_XML: $CHECKSTYLE_XML"
echo "EXCLUDE_FOLDER: $EXCLUDE_FOLDER"

OLD_REVISION=$1
NEW_REVISION=$2
REFNAME=$3

while read OLD_REVISION NEW_REVISION REFNAME; do
    echo
    echo "OLD_REVISION: $OLD_REVISION"
    echo "NEW_REVISION: $NEW_REVISION"
    echo "REFNAME: $REFNAME"
    echo

    # Get the file names, without directory, of the files that have been modified
    # between the new revision and the old revision
    files=`git diff --name-only ${OLD_REVISION} ${NEW_REVISION}`
    #for file in ${files}; do #Para DEBUG
    # echo "File: ${file}"
    #done

    #echo

    # Get a list of all objects in the new revision
    objects=`git ls-tree --full-name -r ${NEW_REVISION}`
    # for obj in ${objects}; do #Para DEBUG
    # echo "Obj: ${obj}"
    # done

    #echo

    # Iterate over each of these files
    for file in ${files}; do

        # Search for the file name in the list of all objects
        object=`echo -e "${objects}" | egrep "(s)${file}$" | awk '{ print $3 }'`

        # If it's not present, then continue to the the next itteration
        if [ -z ${object} ]; then
        continue;
        fi

        # Otherwise, create all the necessary sub directories in the new temp directory
        mkdir -p "${TEMPDIR}/`dirname ${file}`" &>/dev/null
        # and output the object content into it's original file name
        git cat-file blob ${object} > ${TEMPDIR}/${file}

    done;
done

#TODO: só rodar o checkstyle se houver arquivo na pasta? Pois pode ter
#push sem arquivo, como na criação de uma branch
#Exemplo do comando: find ./teste/ -type f | wc -l

#TODO: se o arquivo /src/main/others/TechThingsCool_CheckStyle_Google.xml houver sido comitado,
#ele deve substituir em disco o arquivo $CHECKSTYLE_XML
#Se falhar na substituição do arquivo, o hook deve falhar informando o problema!
#Exemplo do comando: find ./teste/ -type f -name "file.txt" | head -n 1

echo "Começando checagem..."
echo "Rodando checkstyle"
OUTPUT_CHECKSTYLE_FULL=`java -jar $CHECKSTYLE_JAR -c $CHECKSTYLE_XML --exclude "$EXCLUDE_FOLDER" "$TEMPDIR"`
echo "Checagem de código concluída"
echo
echo "Realizando contabilização para resumo"
OUTPUT_CHECKSTYLE_RESUME=`echo "$OUTPUT_CHECKSTYLE_FULL" | grep -P "$CHECKSTYLE_REGEX_WARN" | sed -E "s/$CHECKSTYLE_REGEX_WARN/1/" | sort | uniq -c | sort -rn`
echo "Contabilização concluída"

echo

if [[ -z "${OUTPUT_CHECKSTYLE_RESUME// }" ]]; then #Checa se nulo ou empty removendo os espaços em branco
    BAD_STYLE=0
    echo "Tudo ok com a checagem de estilo de código. Nenhum alerta."
else
    BAD_STYLE=1
    echo "Alerta no estilo de código (veja problemas abaixo)."
    echo "Detalhes:"
    echo "$OUTPUT_CHECKSTYLE_FULL"
    echo
    echo "Resumo:"
    echo " Qtd Alert Description"
    echo "$OUTPUT_CHECKSTYLE_RESUME"
    echo
    echo "Alerta no estilo de código (veja problemas acima)."
fi

echo

if [[ $BAD_STYLE -eq 1 ]]; then
    exit 1
fi
{% endhighlight %}

## Fazendo o script funcionar

- Ter o Java instalado;  
- Colocar dentro da pasta `custom_hooks`, criada anteriormente, o jar do checkstyle e o xml do estilo. Na variáveis `CHECKSTYLE_JAR` e `CHECKSTYLE_XML` defina seus valores, que, para o exemplo, utilizaram checkstyle-7.1-all.jar e TechThingsCool_CheckStyle_Google.xml respectivamente;  
- Realizar um push com e outro sem alerta de estilo de código. Verifique o output do push.

_**Obs:** o arquivo TechThingsCool_CheckStyle_Google.xml pode ser substituído pelo padrão do Google, que pode ser obtido em: [https://github.com/checkstyle/checkstyle/tree/master/src/main/resources](https://github.com/checkstyle/checkstyle/tree/master/src/main/resources)_

**Versões**  
GitLab Community Edition 8.7.2  
Checkstyle 7.1: [Link](http://downloads.sourceforge.net/project/checkstyle/checkstyle/7.1/checkstyle-7.1-all.jar)

**Referências**  
[http://docs.gitlab.com/ce/administration/custom_hooks.html](http://docs.gitlab.com/ce/administration/custom_hooks.html)  
[https://git-scm.com/book/it/v2/Customizing-Git-Git-Hooks](https://git-scm.com/book/it/v2/Customizing-Git-Git-Hooks)  
[http://checkstyle.sourceforge.net/cmdline.html](http://checkstyle.sourceforge.net/cmdline.html)