PEPE THE FROG
=============

![](../images/reto2/reto2.jpg)

Enumeracion de servicios y puertos
----------------------------------

Empezaremos con NMAP y creo que la usaremos mucho :nmap -p- --open T5 -n url vemos el puerto 80 abierto y lo escaneamos un poco mas a fondo.

![](../images/reto2/nmap.jpg)

* * *

Con nmap usando la opción --script http-enum nos reporta que hay una carpeta img, hasta ahí bien abrimos el navegador y con la url/img accedemos a la misma.  
Observamos en el codigo fuente que hay un string en base64,UzRsdjBDMG5EdUN0MCE=.

Ahora accedemos a la carpeta img con el navegador y nos aparece una imagen.La descargamos y ahora tenemos una imagen y un salvoconducto hummm!!.  
Usaremos Steghide para comprobar si hay algo oculto nos pide un salvoconducto se lo damos y premio nos reporta un archivo de texto.

* * *

![](../images/reto2/base64.jpg)

* * *

Acceso a la maquina por SSh
---------------------------

* * *

En nuestra terminal usaremos las credenciales del archivo de texto ssh 10.10.10.XX

Bueno pues ya estamos dentro de la maquina victima y vemos que estamos en la carpeta pepethefrog,  
listamos para ver que hay en la carpeta y nos aparece la primera Flag user.txt,  
listamos otra vez con ls -a para ver archivos y carpetas ocultos y vemos otro archivo que no me cuadra .bash\_none,  
hacemos un cat y zas en toda la boca, la siguiente Flag root.txt.

### Credenciales

usuario: Pepethefrog pass: 4d3l4nt3p3p3

* * *

![](../images/reto2/flags.jpg)

Herramientas utilizadas para este reto:

[**Nmap**](https://nmap.org/)

[**steghide**](http://steghide.sourceforge.net/)

Foro CHE y grupo de telegram
----------------------------

[Comunidad de Hacking Ético](http://ctf.comunidadhackingetico.es/home)

![](../images/logo.jpg)

Podeis pedir ayuda de cualquier reto a la comunidad. [Grupo de Telegram](https://t.me/HackingEticoEs)
