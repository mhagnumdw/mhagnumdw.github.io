Por padrão `o mvn dependency:tree` só analisa projetos (pom.xml) locais. A ideia
aqui é por meio de um GAV (<groupId> <artifactId> <version>) saber suas dependências.

Com base nessa ideia https://stackoverflow.com/questions/22355688/how-to-list-the-transitive-dependencies-of-an-artifact-from-a-repository#22411538

Chega-se nesse script

```shell
#!/bin/sh
if [ "$#" -ne 3 ]; then
  echo "Usage: $0 <groupId> <artifactId> <version>"
  exit
fi

POM_DIR="`echo "$1" | tr . /`/$2/$3"
POM_PATH="$POM_DIR/$2-$3.pom"
echo "POM_DIR: $POM_DIR"
echo "POM_PATH: $POM_PATH"

set -x
mkdir -p "$HOME/.m2/repository/$POM_DIR"
wget -q -O "$HOME/.m2/repository/$POM_PATH" "https://repo.maven.apache.org/maven2/$POM_PATH"
set +x
mvn -f "$HOME/.m2/repository/$POM_PATH" dependency:tree
```

Mas esse script usa um wget pra obter a dependência. Porque não usar o `mvn dependency:get ...` para obtê-la?

Como pode ser visto em
- https://stackoverflow.com/questions/1895492/how-can-i-download-a-specific-maven-artifact-in-one-command-line#1896110
- https://maven.apache.org/plugins/maven-dependency-plugin/usage.html#dependency:get

