# Ambiente
Jenkins 2.32.3 com JDK 8
Versão do Maven no Jenkins: 3.2.1
Projeto para buildar: Hibernate 3.6 com JDK 5

# Motivação
O Hibernate 3.6 necessita da JDK 5 para buildar corretamente. Mesmo adicionando no Jenkins a JDK 5, o maven 3.1.1 que é a última versão compatível para essa JDK e configurando o job para fazer uso dessas versões, teremos o problema: _Exception in thread "main" java.lang.UnsupportedClassVersionError: Bad version number in .class file_

# Configurando o toolchain / maven-toolchains-plugin no Jenkins e no projeto

## Jenkins

- instalar o plugin: [Config File Provider Plugin](https://wiki.jenkins.io/display/JENKINS/Config+File+Provider+Plugin)

### Criar configuração do toolchain
Jenkins > Manage Jenkins > Managed files > Add a new Config > Maven toolchains.xml > Submit

Name: Hibernate-Toolchains

Content:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<toolchains>
  <toolchain>
     <type>jdk</type>
     <provides>
         <version>1.5</version>
         <vendor>oracle</vendor>
     </provides>
     <configuration>
        <jdkHome>/usr/java/jdk1.5.0_22</jdkHome>
     </configuration>
  </toolchain>
</toolchains>
```

Clicar em Submit

Então será redirecionado para todos os arquivos de configuração.

Obs: a configuração acima apenas indica onde está o binário
da jdk da Oracle versão 1.5 para um projeto que esteja
configurado com o plugin maven-toolchains-plugin.


### Configurar o job

JDK 8

Git: repo do hibernate
Branches to build: 3.6

Na seção 'Build Environment'
Mecar o checkbox 'Provide Configuration files'
E preencher:
File: Hibernate-Toolchains
Variable: TOOLCHAIN

Na seção 'Build'
Em 'Goals and options' referenciar a configuração assim: -t $TOOLCHAIN

Abaixo um exemplo completo de Goals:
`-t $TOOLCHAIN -B -V clean package verify -DskipTests -Dmaven.test.skip=true -DdisableDistribution=true -Djdk16_home=/usr/lib/jvm/java-1.7.0-oracle-1.7.0.75-1jpp.2.el7.x86_64`


### Configurar o projeto

Configuração no pom.xml do projeto (snippet)

```xml
<build>
    <plugins>
    	<!-- indica que deve compilar com jdk 5 da oracle -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-toolchains-plugin</artifactId>
            <version>1.1</version>
            <executions>
                <execution>
                    <goals>
                        <goal>toolchain</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <toolchains>
                    <jdk>
                        <version>1.5</version>  
                        <vendor>oracle</vendor>
                    </jdk>
                </toolchains>
            </configuration>
        </plugin>
    </plugins>

	<pluginManagement>
	    <plugins>
	        <plugin>
	            <groupId>org.apache.maven.plugins</groupId>
	            <artifactId>maven-toolchains-plugin</artifactId>
	            <version>1.1</version>
	        </plugin>
	    </plugins>
	</pluginManagement>
</build
```

# É isso
A partir de então, mesmo que o maven do Jenkins seja invocado com a jdk 8, o build do Hibernate vai acontecer com a jdk 5.

# Referências
https://stackoverflow.com/questions/46276236/jenkins-error-when-i-try-to-use-an-older-jdk-for-a-specific-maven-project?rq=1
http://maven.apache.org/plugins/maven-toolchains-plugin/toolchains/jdk.html
https://support.cloudbees.com/hc/en-us/articles/115003910392-How-to-config-a-Maven-Toolchain
https://maven.apache.org/docs/history.html

