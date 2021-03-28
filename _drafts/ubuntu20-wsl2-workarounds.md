# SNAP

Comandos como `snap list` e `snap version` falham com a mensagem de erro:

`error: cannot communicate with server: Post http://localhost/v2/snaps/hello-world: dial unix /run/snapd.socket: connect: no such file or directory`

Workaround:

```shellscript
sudo apt-get update && sudo apt-get install -yqq daemonize dbus-user-session fontconfig
sudo daemonize /usr/bin/unshare --fork --pid --mount-proc /lib/systemd/systemd --system-unit=basic.target
exec sudo nsenter -t $(pidof systemd) -a su - $LOGNAME

snap version
```

Observações:
- Ao executar em um shell, não funciona em outro shell
- Para executar em outro shell precisa parar o wsl: `wsl --shutdown`
- Podem existir efeitos colaterais diversos (qualquer coisa é parar o wsl)

Referências:
- https://github.com/microsoft/WSL/issues/5126#issuecomment-653715201-permalink

# Outro...
