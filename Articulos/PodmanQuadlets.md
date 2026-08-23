---
title: "Podman y los quadlets: gestiona contenedores con systemd y qctl"
date: 2026-08-23
author: h1dr0
tags: [podman, quadlets, systemd, contenedores, qctl, devops, atareao]
category: Articulo
description: "Aprende qué son los Podman quadlets y cómo gestionarlos con qctl: instala, arranca y monitoriza tus contenedores como servicios systemd de forma sencilla."
keywords: podman quadlets, qctl, podman systemd, contenedores rootless, quadlets ejemplo
---

## Qué son los quadlets y por qué deberían importarte

Si llevas un tiempo con Podman, seguro que has oído hablar de los *quadlets*. En pocas palabras, un quadlet es un archivo de definición —con extensiones como `.container`, `.volume`, `.network`, `.pod`, `.kube`, `.build`, `.image` o `.artifact`— que Podman sabe convertir en una unidad de systemd "de verdad" mediante un generador llamado `podman-system-generator`.

La idea es elegante: en lugar de lanzar contenedores a mano con comandos largos y propensos a olvidarse, escribes un archivo de texto plano que describe *qué* quieres ejecutar. Cuando el sistema arranca (o cuando tú haces `systemctl daemon-reload`), Podman lee esos archivos y genera los `.service` correspondientes, que luego se gestionan con `systemctl` como cualquier otro servicio del sistema.

Esto convierte a los contenedores en **ciudadanos de primera clase de systemd**. Y eso trae ventajas de las que systemd ya se encarga muy bien: arranque automático al iniciar el equipo, dependencias con `After`/`Requires`, políticas de reinicio y registros accesibles vía `journalctl`. Todo sin que tengas que aprenderte de memoria la sintaxis completa de `podman run`.

En el caso de una instalación *rootless* (sin privilegios de administrador), la ruta típica donde se leen estos archivos es `~/.config/containers/systemd/`. Si fueras root, sería `/etc/containers/systemd/`.

## El problema: hacerlo a mano se vuelve tedioso

Vale, los quadlets suenan genial. Pero hay un "pero". Gestionarlos manualmente implica varios pasos repetitivos: colocar el archivo en la carpeta correcta, crear los *symlinks* adecuados, recordar ejecutar `systemctl --user daemon-reload` cada vez que cambias algo, arrancar el servicio, comprobar el estado, mirar los logs... y repetir todo el ciclo cuando actualizas la imagen o reinstalas.

Nada de eso es difícil, pero es mecánico y propenso a olvidos. Aquí es donde entra **qctl**, una utilidad que se propone justamente quitarte esas tareas de encima.

## Qué es qctl (y cómo instalarlo)

`qctl` es una herramienta escrita en Rust, creada por el proyecto *atareao*, pensada para instalar y gestionar quadlets y tareas de forma cómoda desde la línea de comandos. Puedes ver su código en <https://github.com/atareao/qctl>.

Un punto importante: **no hay script de instalación, ni paquete pip, ni flatpak documentado**. La vía oficial que aparece en su documentación es compilarlo tú mismo con `cargo`. No suena tan cómodo como un `apt install`, pero es un proceso directo si ya tienes las herramientas de desarrollo.

Los requisitos son sencillos:
- Un sistema Linux.
- Tener instalado `cargo` (el gestor de paquetes y compilador de Rust).
- Disponer de `systemctl --user`.
- Tener `podman` y `journalctl` disponibles.

Para instalarlo, sigues estos pasos:

```bash
git clone https://github.com/atareao/qctl
cd qctl
cargo build --release
```

Tras la compilación, el binario queda en `./target/release/qctl`. Ese es el ejecutable que usarás en todos los ejemplos.

## Ejemplo paso a paso: desplegar un nginx con qctl

Vamos a verlo con un ejemplo práctico y didáctico. Supongamos que queremos levantar un servidor web nginx usando un quadlet y gestionarlo con qctl.

**Paso 1: crear el archivo de definición.**

Creamos un archivo llamado `myapp.container` con este contenido:

```ini
[Unit]
Description=My app

[Container]
Image=docker.io/library/nginx:alpine
PublishPort=8080:80

[Install]
WantedBy=default.target
```

Fácil de leer, ¿verdad? Le decimos a Podman qué imagen usar (`nginx:alpine`), que publique el puerto `8080` del host hacia el `80` del contenedor, y con `WantedBy=default.target` indicamos que queremos que arranque con el sistema.

Una forma rápida de crearlo desde el terminal:

```bash
cat > myapp.container <<'EOF'
[Unit]
Description=My app
[Container]
Image=docker.io/library/nginx:alpine
PublishPort=8080:80
[Install]
WantedBy=default.target
EOF
```

**Paso 2: compilar qctl** (si aún no lo has hecho):

```bash
cargo build --release
```

**Paso 3: instalar el quadlet.**

```bash
./target/release/qctl install
```

Aquí ocurre la magia silenciosa: `qctl` crea los *symlinks* necesarios en `~/.config/containers/systemd/` y luego ejecuta `systemctl --user daemon-reload`, para que systemd regenere las unidades a partir de tu archivo. qctl busca los archivos .container en el directorio actual (o en ./quadlets/) y crea los symlinks en ~/.config/containers/systemd/.

**Paso 4: arrancar el servicio.**

```bash
./target/release/qctl start myapp
```

Este comando inicia el servicio a través de systemd, así que el contenedor se levanta como un servicio más del sistema.

**Paso 5: comprobar el estado.**

```bash
./target/release/qctl status
```

Te muestra el estado de los servicios gestionados. Si quieres algo más compacto, la herramienta admite `qctl status --compact`.

**Paso 6: ver los logs.**

```bash
./target/release/qctl logs myapp
```

Por detrás, esto no es más que un `journalctl --user -u myapp -f --since "1 hour ago"`, pero te ahorra teclear toda la ristra y te deja ver la salida del contenedor al instante.

## Repaso de comandos útiles

Aquí tienes los comandos que documenta qctl y para qué sirven:

- `install`: crea los symlinks en `~/.config/containers/systemd/` y hace `systemctl --user daemon-reload`.
- `uninstall`: elimina lo instalado.
- `reinstall`: reinstala (útil tras cambiar el archivo).
- `start [SERVICE]` / `stop [SERVICE]` / `restart [SERVICE]`: controlan el ciclo de vida del servicio.
- `update [SERVICE]`: hace un `podman pull` de la imagen definida en `Image=`.
- `status [SERVICE] [--compact]`: muestra el estado.
- `clean-volumes`: limpia volúmenes.
- `check <QUADLET>`: valida el archivo usando `/usr/lib/podman/quadlet -dryrun -user`.
- `logs <SERVICE>`: muestra los logs vía `journalctl`.
- `reload`: ejecuta `systemctl --user daemon-reload` (existe en el código aunque no aparece en el README, y es práctico cuando solo quieres regenerar las unidades).

> Nota: el comando `logsf` aparece mencionado en el README pero **no está implementado**, así que mejor no contar con él por ahora.

## Ventajas de usar qctl + quadlets

Combinar ambos te da lo mejor de los dos mundos. Por un lado, los quadlets te dan integración nativa con systemd: arranque automático con `WantedBy=default.target`, dependencias declaradas con `After`/`Requires`, reinicios gestionados por el propio systemd y logs centralizados en `journalctl`. Por otro, qctl te quita la fricción de tener que crear symlinks a mano, recordar el `daemon-reload` y memorizar las rutas.

El resultado es un flujo de trabajo reproducible: defines tu contenedor una vez en un archivo de texto, lo instalas con un comando y lo olvidas, sabiendo que systemd se encarga de mantenerlo vivo.

## Cierre

Los quadlets son, probablemente, la forma más "linuxera" y ordenada de ejecutar contenedores con Podman: aprovechan la herramienta de init que ya usas cada día. Y aunque gestionarlos a mano es perfectamente posible, herramientas como **qctl** demuestran que un poco de automatización puede convertir una tarea repetitiva en algo tan simple como `install`, `start` y `logs`. Si te mueves en entornos rootless y quieres que tus contenedores se comporten como servicios de verdad, vale la pena darle una oportunidad a este dúo.
