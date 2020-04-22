Windows WSL - Alterar permissões com chmod não surte efeito

# Problema

cd /tmp/
echo "teste" > teste.txt
ls -la

aqui vão aparecer as permissões

tentar mudar a permissão:

chmod 600 teste.txt
ls -la

aqui vai dá pra ver que as permissões do arquivo não mudaram

Antes de irmos para a solução vamos guardar o output do comando `mount -l > /tmp/mount-antes.txt`


# Solução

Dentro do WSL Linux, no caso aqui um WSL Ubuntu, editar/criar o arquivo
/etc/wsl.conf
Com o conteúdo:
[automount]
options = "metadata"

Sair do WSL

Abrir o power shell

Obter o nome da instância WSL que foi alterada: `wsl --list`

O output do comando pode ser algo parecido com
```
Distribuições do Subsistema do Windows para Linux:
Ubuntu (Padrão)
```

Desligar a instância pelo nome
wsl -t Ubuntu


# Verificando se tudo ok

Abrir o WSL Ubuntu
- Executar o chmod no arquivo e verificar se as permissões mudaram

Pra finalizar vamos gerar um novo output do mount: `mount -l > /tmp/mount-depois.txt`

Você teve ter um diff parecido com o diff abaixo. Observar o atributo `metadata` na diferença.

Ver arquivo: windows-wsl-linux-chmod-nao-funciona-diff.png

# Referências

- https://github.com/microsoft/WSL/issues/3181#issuecomment-394762108
- https://devblogs.microsoft.com/commandline/automatically-configuring-wsl/
- https://superuser.com/questions/1126721/rebooting-ubuntu-on-windows-without-rebooting-windows#1347725

