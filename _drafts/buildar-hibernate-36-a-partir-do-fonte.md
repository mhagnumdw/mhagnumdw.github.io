Ambiente onde esse passo a passo foi realizado:
- Fedora 29

Opcional:
Testar esses passos em um docker: docker run -t -i fedora bash

# instalar algumas dependências necessárias e úteis
dnf install git bash-completion iputils coreutils findutils which libnsl xmlstarlet wget vim

# clona o repositório oficial
git clone https://github.com/hibernate/hibernate-orm.git

# listar as branches remotas
git branch -r

# alternar para branch "3.6"
git checkout 3.6

# apenas verificar a versão
# versão do pom.xml está em: 3.6.11-SNAPSHOT

# necessário jdk 5 e 6, mas com a 6 não funcionou
# no lugar da jdk 6 usar a jdk 7
# https://www.oracle.com/technetwork/java/archive-139210.html
# baixar jdk 5: jdk-1_5_0_22-linux-amd64.bin
# baixar jdk 7: jdk-7u80-linux-x64.rpm

# instalar jdk 7
# instalar
rpm -Uvh jdk-7u80-linux-x64.rpm
# verificar versão
java -version

# instalar jdk 5
# instalar
./jdk-1_5_0_22-linux-amd64.bin
# verificar versão
~/jdk1.5.0_22/bin/java -version

# gerar release
# poderia ser pelo plugin maven-release-plugin do maven, mas em razão da estrutura
# do projeto do hibernate essa abordagem não funciona. Vamos gerar então
# com um script shell disponibilizado dentro da pasta do projeto.

# entrar na pasta do hibernate
cd hibernate-orm

# criar uma nova branch
git checkout -b 3.6.SEFIN 3.6

# configurar user e email dos commits
git config --local user.name "mhagnumdw" && git config --local user.email "email@email.gov.br"

# gerar release
# ao final o script pede pra fazer o push, como estamos
# fazendo no repositório do GitHub do Hibernate e não temos
# acesso, bas então cancelar: ctrl + c
./tagRelease.sh -r 3.6.11.SEFINv2 -d 3.6.11.SEFINv3-SNAPSHOT

# ver a tag criada
git tag | grep -P '^3.6.'

# ver a diferença entre a branch 3.6 e a branch 3.6.SEFIN (essa última é a que
estamos trabalhando)
git log --oneline origin/3.6..HEAD

# é preciso baixar o maven compatível com jdk 5
# tabela de compatibilidade: https://maven.apache.org/docs/history.html
# baixar
wget https://archive.apache.org/dist/maven/maven-3/3.1.1/binaries/apache-maven-3.1.1-bin.tar.gz
# descompactar
tar -xvzf apache-maven-3.1.1-bin.tar.gz

# compilar / empacotar / deploy

# usar um settings.xml apropriado, que tenha os repositórios maven do JBoss
# e que tenha acesso ao nexus caso seja feito deploy
# ver exemplo de settings em [4]
scp f0692311@d20658:/home/f0692311/ambiente-grpfor/maven/settings-sefin-grpfor-jenkins.xml ~/

# apenas empacotar (não vai gerar o source pois o maven-source-plugin está definido para rodar no verify, ver pom.xml)
# executar
JAVA_HOME="/root/jdk1.5.0_22" /root/apache-maven-3.1.1/bin/mvn -B -V -gs /root/settings-sefin-grpfor-jenkins.xml -DskipTests=true -Dmaven.test.skip=true clean package -DdisableDistribution=true -Djdk16_home=/usr/java/jdk1.7.0_80
# verificar os jars gerados
find | grep -P -i '\.jar$'

# install
# executar
JAVA_HOME="/root/jdk1.5.0_22" /root/apache-maven-3.1.1/bin/mvn -B -V -gs /root/settings-sefin-grpfor-jenkins.xml -DskipTests=true -Dmaven.test.skip=true clean install -DdisableDistribution=true -Djdk16_home=/usr/java/jdk1.7.0_80
# verificar os jars gerados
find /root/.m2/repository/org/hibernate/ | grep -P -i '\.jar$'

# deploy no Nexus
# necessário mudar o <distributionManagement> do ./hibernate-parent/pom.xml
# o settings.xml tem que ter as credenciais para escrever no Nexus
# executar
JAVA_HOME="/root/jdk1.5.0_22" /root/apache-maven-3.1.1/bin/mvn -B -V -gs /root/settings-sefin-grpfor-jenkins.xml -DskipTests=true -Dmaven.test.skip=true clean deploy -DdisableDistribution=true -Djdk16_home=/usr/java/jdk1.7.0_80

# Referências
[1] https://developer.jboss.org/thread/199389#jive-1168031045068142817703
[2] https://hibernate.atlassian.net/browse/HHH-7682
[3] https://github.com/hibernate/hibernate-orm/pull/393
[4] https://developer.jboss.org/wiki/BuildingHibernateFromSource35
[5] https://www.oracle.com/technetwork/java/archive-139210.html
[6] https://bugzilla.redhat.com/show_bug.cgi?id=1653638


