# 🔐 Bandit: niveles 20-33

## 💻 **BANDIT19 → BANDIT20**
<div id="bandit20">
Para conseguir la contraeña al siguiente nivel, debemos utilizar el ejecutable **bandit20-do**, situado en el directorio personal del usuario. Este ejecutable tiene la característica especial de tener activado el **`permiso +s (setuid)`**.

### Modo +s (setuid)
Setuid (set user ID, suid) se refiere a un permiso especial que se le puede conceder a los ficheros de un sistema Linux para que estos al ser ejecutados se comporten como si el que estuviese ejecutando el fichero fuese el dueño del fichero. Es decir, **realizan un cambio de UID al usuario** que ejecuta mientras dura la ejecución del fichero, cambiándolo por el del dueño del fichero.

en concreto este **bandit20-do**, como su nombre indica, permite ejecutar cualquier comando como si el usuario que lo ejecutase fuera bandit20. 

> Nota: Podemos ver este permiso representado con la letra _s_ al ejecutar lo siguiente:
>
>```console
>bandit19@bandit:~$ ls -lah bandit20-do
>-rwsr-x--- 1 bandit20 bandit19 ......... bandit20-do
>```
>
>Como se puede ver, el dueño del fichero es bandit20, por lo que todo comando ejecutado a través de este fichero será como si lo ejecutara el usuario bandit20

Como ejemplo de funcionamiento de esto, podemos probar a ejecutar el comando **`whoami`** que nos dice qué usuario somos dentro del sistema con este ejecutable suid.

```console
bandit19@bandit:~$ ./bandi20-do whoami
bandit20
```

La salida nos dice que somos el usuario bandit20 🏴‍☠️.

Sabiendo esto, podemos obtener la contraseña de forma fácil. Recordar que la contraseña de cada nivel se encuentra en **`/etc/bandit_pass/bandit{nivel}`**


**Solución:**

La manera más sencilla de resolver este nivel es imprimir el contenido del fichero de contraseña de bandit20 usando el ejecutable suid.

```sh
./bandit20-do cat /etc/bandit_pass/bandit20
```

---
</div>
## 💻 **BANDIT20 → BANDIT21**

En este nivel aprenderemos a utilizar la herramienta **netcat** en modo LISTEN, para conseguir la password que habilita el siguiente nivel debemos emplear el binario setuid **`suconnect`** que se encuentra en el directorio. Este programa realiza lo siguiente: establece una conexión a localhost en un puerto dado. 

Es importante entender que este programa actuará como **_cliente_**, por lo que nosotros deberemos crear la conexión que escuche, es decir la parte del **_servidor_**. Esto se puede lograr con la herramienta netcat de Linux y la opción -l.

**Solución**

En una terminal:
```sh
nc -lvp 9000
```
**_Opciones:_**

- **-l**: Listen, permite establecer una conexión de escucha.
- **-v**: Modo verbose, es útil incluirlo ya que otorga más información si ocurre algún error.
- **-p**: Port, para especificar el puerto de escucha.


Luego establecemos la conexión con ./suconnect.
Usamos el puerto 9000 pero cualquier puerto libre es válido

```sh
./suconnect 9000
```

E introducimos la contraseña **en la terminal donde tenemos ejecutando netcat**.

> **NOTA:** Para este nivel es epecialmente útil conocer herramientas que permitan dividir la terminal como **`screen`** o **`tmux`**, esta última muy recomendable aprenderla.

## 💻 **BANDIT21 → BANDIT22**

En este nivel se tiene que existe un programa el cual se ejecuta en el sistema a intervalos regulares de tiempo a través del proceso **`cron`**, y debemos buscar en el directorio **/etc/cron.d/** para ver la configuración de cron para la ejecución de dicho programa, y poder ver qué hace exactamente.

###  Proceso Cron 🕑 ⏳
En Linux, el proceso **`cron`** se utiliza para automatizar tareas y ejecutar programas en intervalos específicos de tiempo. Se pueden establecer intervalos de minutos, horas, días, y hasta qué día concreto de qué mes, por lo que otorga bastante flexibilidad.

Un ejemplo de fichero **_crontab_** en un sistema puede ser el siguiente. En este fichero, se espeficia el programa/script, y a qué intervalos debe crontab ejecutar dicho programa/script. El usuario root (administrador) también puede especificar el usuario que "ejecutará" dicha tareas (con qué permisos básicamente).

Un asterisco en una de las columnas de tiempo indica básicamente "todo". Cinco asteriscos supondría que se ejecuta cada minuto:

```bash
# Minuto  Hora  Día del Mes  Mes  Día Semana  Usuario Comando
0 3 * * * root /usr/bin/backup.sh     # Backup diario a las 3 AM
0 0 1 * * root /usr/bin/monthly_report.sh  # Genera un informe el primer día de cada mes a medianoche
```
#### Crontab del sistema VS crontab de usuarios:
Además del fichero **`/etc/crontab`**, cada usuario puede tener su propio fichero de crontab que se ejecutará con sus permisos, pero esto no aplica a este problema.

En este nivel en lugar de usar un fichero **crontab** se emplean varios ficheros dentro del directorio **/etc/cron.d**. Listando dicho directorio encontramos un fichero llamado cronjob_bandit22, el nombre implica que nos interesa.

**Solución:**

El contenido del fichero de cron es el siguiente. Nos muestra que hay un .sh que se ejecuta "silenciosamente" cada minuto:

```bash
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null 
```

> **NOTA:** *&> /dev/null* indica que no se muestre la salida proporcionada en terminal

Si hacemos un less al contenido de ese fichero veremos que lo que hace el .sh es copiar la contraseña del nivel actual a un fichero dentro de la carpeta temp:

```sh
#!/bin/bash
chmod 644 /tmp/xxxx
cat /etc/bandit_pass/bandit22 > /tmp/xxx..
```

Por lo que para obtener la contraseña simplemente deberemos inspeccionar dicho fichero, ya que tenemos los permisos necesarios gracias al chmod 644 ejecutado en el shellscript .sh.

```sh
cat /etc/xxx..
```

---

## 💻 **BANDIT22 → BANDIT23**

En este nivel el problema es bastante similar al nivel anterior. Debemos mirar en la carpeta /etc/cron.d para ver el cronjob que se ejecuta, esta vez, como usuario bandit23:

```bash
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh &> /dev/null 
```

El shellscript al que apunta ahora es algo más complejo:

```sh
#!/bin/bash
#Contenido de /usr/sbin/cronjob_bandit23.sh
myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

Este shellscript hace varias cosas:
1. Guarda el nombre del usuario que ejecuta el script (bandit23) en la variable de entorno _$myname_
2. Hace una operación hash y guarda el resultado en la variable _$mytarget_
3. Copia el fichero de contraseña **que necesitamos** a _/temp/$mytarget_.

Lo que debemos hacer entonces es replicar este comportamiento para descubrir el valor de _**$mytarget**_ y ver dónde se guarda la contraseña.

>NOTA: debemos hacerlo asignando el valor **bandit23** a $myname, si no el hash md5 saldrá diferente no obtendremos el valor que queremos.

```sh
bandit22@bandit:~$ $myname=bandit23
```
```sh
bandit22@bandit:~ $mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
bandit22@bandit:~ $echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget

```
Obtenemos así el nombre y la localización del fichero donde se copia la contraseña

```sh
cat /etc/8ca319....
```

---

## 💻 **BANDIT23 → BANDIT24**

En este nivel seguimos teniendo que trabajar con cron, al igual que en los dos anteriores. Seguimos teniendo un fichero **`/etc/cron.d/cronjob_bandit24`**

```bash
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null 
```

Ahora el shellscript es diferente:

![Cap2.jpg](Cap2.jpg)

Lo que hace este script es ejecutar todos los ficheros dentro de la carpeta **`/var/spool/$myname/foo`**, siendo **_$myname_** bandit24 (el usuario), y posteriormente eliminarlos. Todos los ficheros que se ejecuten lo harán con los permisos de bandit24.

Sabiendo esto la solución será crear un shellscript sencillo que nos permita de alguna forma leer la contraseña almacenda en **`/etc/bandit_pass/bandit24`** (recordar que todas las contraseñas de los niveles se almacenan en ese directorio). POr ejemplo, podemos crear una carpeta temporal con **mktemp -d** y guardar allí una copia de la contraseña para poder leerla.

**Solución:**

```sh
#!/bin/bash
less /etc/bandit_pass/bandit24 > /tmp/tmp.xxxxxx/thePass
```
Para que bandit24 mediante cron pueda ejecutar este fichero, necesitaremos darle permisos generosos al shellscript y al directorio temporal. Luego, copiarlo al directorio **`/var/spool/bandit24/foo`**.

```console
chmod 777 myBashscript.sh
chmod 777 /tmp/tmp.xxxx..
cp myBashScript.sh /var/spool/bandit24/foo
```

Al cabo de unos segundos, debería aparecer el fichero /tmp/tmp.xxxxxx/thePass con la contraseña del siguiente nivel.

---

## 💻 **BANDIT24 → BANDIT25**

Hay un _daemon_ (programa servidor) corriendo en el puerto 30002, el cual, si se le proporciona la contraseña de este nivel junto con un **``pin de 4 dígitos``**, nos proporcionará la contraseña al siguiente nivel. La única forma de obtener el pin correcto es mediante ***fuerza bruta** es decir, probar las 10000 combinaciones posibles (0000-9999).

Como introducirlas a mano sería enormemente ineficiente, lo mas óptimo es escribir un shellscript para pdoer probar las 10K combinaciones de códigos. Para no abrir un nuevo socket cada vez que probemos una clave, usaremos el shellscript para crear un **diccionario**.

**Solución**

Creo el siguiente shellscript para escribir un diccionario de 10000 combinaciones posibles para posteriormente pasarlo línea por línea a través del socket mediante netcat.

```sh
#!/bin/bash

file=$/etc/bandit_pass/bandit24
for i in {0000..9999}
do
    echo $(cat $file) $i >> dict.txt #password y clave deben ir separados por un whitespace
done
```

>**NOTA:** Usamos >> en lugar de > para no sobreescribir el archivo en cada ejecución del bucle

Este fichero nos crea un diccionario de 10000 líneas. Si inspeccionamos las primeras líneas veremos algo así:

![cap3.jpg](cap3.jpg)

Para conectar con el servicio del puerto 30002 usamos la herrramienta netcat. Podemos usar cat primero y redirigir la salida del fichero al socket con pipe (|).

```console
cat dict.txt | nc localhost 30002
```

Esto probará todas las líneas del diccionario hasta que aparezca la commbinación correcta. Entonces se nos mostrará la contraseña al siguiente nivel.

---

## 💻 **BANDIT25 → BANDIT26**

En el directorio home del usuario bandit25 encontramos el fichero **`bandit26.sshkey`**, el cual nos permite acceder via ssh al siguiente nivel como el usuario bandit26 y así poder leer la contraseña del usuario de una forma fácil, al igual que en el nivel de [Bandit23](./Bandit9-19.md). Sin embargo, alguien ha modificado la **shell** del usuario Bandit26 para que no sea _/bin/bash_, por lo que no podremos ejecutar comandos con este usuario a menos que seamos capaces de establecer la shell correcta.

> Este nivel requiere un conocimiento muy específico de las herramientas **`more`** y **`vim`** y algunas de sus opciones.

###  Shell en Linux 💻🐚

En los sistemas GNU/Linux, la shell es el programa que hace de intermediario entre el usuario que tecleaa los comandos y el kernel del sistema operativo. La shell más común en Linux es **bash** (Bourne Again Shell), aunque existen otras como fish shell o Z Shell. 

Cada usuario tiene una shell configurada. Sin una shell, el usuario no podrá ejecutar comandos en al terminal.

**Solución:**

Si intentamos acceder con la clave de ssh para bandit26, el sistema nos expulsa al instante. Tampoco nos deja ejecutar comandos remotamente con ssh. Debemos por tanto cambiar la shell del usuario bandit26.

Para ver la shell del usuario bandit26 desde bandit25, inspeccionamos el contenido del fichero **`/etc/passwd`** y filtramos con grep. El último campo de la tupla nos indica la shell del usuario. 

```she
cat /etc/passwd | grep "bandit26"
...
bandit26:x:10026:10026:bandit level 26:/home/bandit26:/usr/bin/showtext
```

Parece que en lugar de /bin/bash, la shell del usuario es **`/usr/bin/showtext`**. Inspeccionamos ese fichero para ver qué es.

![Cap4.jpg](Cap4.jpg)

Lo que hace esta shell es ejecutar el comando **more** sobre un fichero ubicado en la carpeta home del usuario bandit26, el cual no tenemos permisos para leer ni escribir como usuario bandit25. El comando more inspecciona el contenido de un fichero y (inportante), **si el contenido es demasiado extenso para entrar en el tamaño de nuestra terminal, more permite hace scroll y realizar búsquedas**, entre otras cosas. Es muy útil por tanto para leer ficheros largos.

Una de las utilidades de more es que incorpora el editor de textos **vim** pulsando la tecla _v_, permitiendo editar el fichero que se está leyendo.

El editor vim tiene muchas opciones, entre las cuales están la opción **:e** que permite visualizar ficheros (como cat o less).

Por tanto, si establecemos la conexión mediante ssh usando la clave, y reescalamos la terminal para que sea lo suficientemente pequeña podremos hacer que el **more** se ejecute "modo scroll".

Después, podemos pulsar _v_ para abrir **vim** como usuario bandit26, y a partir de ahí con la opción :e /etc/bandit_pass/bandit26, obtener la contraseña del nivel.

### **Otra solución posible**

Vim también tiene otra opción interesante que es **:set**, la cual permite establecer diferentes ajustes para esa ejecución concreta de vim. Otra de las opciones es **:shell**, la cual permite abrir una instancia de la shell del usuario. 

Con **:set shell** por tanto podemos establecer la shell de vim como /bin/bash, y después con :shell abrir una instancia de bash, donde ya podríamos ejecutar comandos con normalidad.

---

## 💻 **BANDIT26 → BANDIT27**

Habiendo obtenido acceso a una shell decente como usuario bandit26, conseguir la contraseña de bandit27 es una tarea sencilla. La tarea es la misma que en el nivel [Bandit20](#bandit20).

**Solución**

El fichero **`bandit27-do`** de la carpeta home del usuario bandit26 tiene el bit +s activado, y nos permite ejecutar comandos como el usuario bandit27. POr tanto podemos usarlo para leer el contenido de la contraseña del usuario bandit27.

```sh
./bandit27-do cat /etc/bandit_pass/bandit27
```

---

## 💻 **BANDIT27 → BANDIT28**

hay un repositorio de git en la url ssh://bandit27-git@localhost/home/bandit27-git/repo en el puerto 2220. Debemos clonarlo y buscar la información de este repositorio para encontrar la contraseña al siguiente nivel.

### **Git y GitHub**
Git es una herramienta de control de versiones my ampliamente usada en desarrollo e ingeniería software. Permite mantener un histórico de versiones, crear ramas, producir versiones, regresar a versiones anteriores par revertir cambios, blabla... La herramienta para manejar Git por línea de comandos es **``git``**. Por ejemplo, con el comando **`git init`** podemos inicializar un nuevo repositorio y comenzar a trabajar.

Por otro lado está GitHub, que cumple la misma función que Git, pero a diferencia de Git que funciona en local,  GitHub es un servicio en la nube.

Lo más común en desarrollo software es tener un repositorio local de Git y sincronizarlo con un repositorio remoto de GitHub, lo que permite que muchas personas trabajen en el mismo repositorio de forma colaborativa.
Para esto se usan comandos como **``git clone``** que sirve para clonar un repositorio de GitHub (u otro servicio de repositorio en la nube como GitLab) y **``git push``** o **``git pull``** para subir/descargar el contenido de una rama específica del repositorio local y remoto (por ejemplo, para subir a producción una nueva versión de un software).

**Solución**

La solución a este nivel es sencilla. Basta con clonar el repositorio y buscar un poco en su interior. Clonamos con el siguiente comando. 

```sh
git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo

#Nos solicitará una contraseña, la misma que para el nivel actual
```

Esta acción descarga el repositorio remoto a un repositorio local de git. Si buscamos dentro de la carpeta repo que nos aparece, encontramos un archivo *readme* el cual contiene la contraseña buscada.

---

## 💻 **BANDIT28 → BANDIT29**

En este nivel, la problemática es similar al nivel previo. tenemos un repositorio de git en _**ssh://bandit28-git@localhost:2220/home/bandit28-git/repo**_, que debemos clonar.

Clonamos con **git clone** y encontramos el mismo archivo README, pero ahora la contraseña no aparece:

```
## credentials

- username: bandit29
- password: xxxxxxx
```

**Solución**

Intuimos que quizás la contraseña estuvo ahí alguna vez. para ver el historial de cambios de un repositorio empleamos el siguiente comando desde dentro del repositorio:

```sh
git log
```
Este comando muestra el histórico de **``commits``** de un repositorio. Un commit es un punto de guardado del repositorio, representado por un valor **hash** y por un **mensaje de texto**.

Al ejecutar el comando, podemos ver desde el último commit realizado (representado por el puntero **``HEAD->[nombre de la rama])``**), hasta el primero de los commits:

![alt text](image.png)

Vemos que hay un commit específico cuyo mensaje es "fix info leak". Tiene pinta de ser lo que estamos buscando. Si conseguimos ver una versión anterior a ese commit, quizás encontremos la contraseña en el README.

para volver a un commit específico, necesitamos el HASH que identifica ese commit, y ejecutar el comando:

```bash
git checkout {commit_hash}
```

Una vez nos hemos movido a una versión anterior podremos encontrar la contraseña inspeccionando el fichero README.

---

## 💻 **BANDIT29 → BANDIT30**

Objetivo similar a los dos niveles anteriores. tenemos un repositorio de git en _**ssh://bandit29-git@localhost:2220/home/bandit29-git/repo**_, que debemos clonar.

Como en los dos casos anteriores, clonamos usando git clone y la clave ssh del nivel, e inspeccionamos el único fichero que hay (README.md):

```
## credentials

- username: bandit30
- passwords: no passwords in production!
```

En esta ocasión el mensaje nos indica que no se muestra la contraseña debido a que "no se permiten contraseñas en producción".

### Git branching 🌿
En repositorios de Git y GitHub, lo más común es organizar el desarrollo y las versiones en **``branches``** (ramas), donde se tiene una rama **main** (principal), que muchas veces es llamada rama de producción, ya que contiene una versión funcional del software; y por otro lado se tienen otras ramas donde cada rama implementa una _feature_ (funcionalidad), o es usada por un único desarrollador, dependiendo del proyecto y de las decisiones del equipo.

Cada rama tiene su propio histórico de commits. Es posible fusionar ramas en otra rama. Esto es muy común por ejemplo, cuando se tiene una rama que implementa una funcionalidad nueva, y se quiere añadir esa funcionalidad a el resto del sistema. Gráficamente sería algo como:

```sh
          A---B---C topic branch
         /         \
    D---E---F---G---H   production branch

# En el punto 'A' se creó una rama nuev (git checkout -b topic)
# En el punto C se fusionó la rama topic con la rama production (git merge)

```
**Solución**

la conclusión es que debemos buscar si el repositorio tiene otras ramas donde podamos buscar. Con el comando **``git branch -a``** listamos todas las ramas (locales y remote-tracking).

>Las ramas remote-tracking son unas ramas especiales que repreentan una copia de una rama de un repositorio remoto vinculado al local.

Si ejecutamos el comando vemos que aparecen varias ramas de tipo remote-tracking (prefijo remotes/origin...). La rama remotes/origin/dev parece prometedora, ya que el mensaje decía que nada de contraseñas en producción, quizás en ramas de desarrollo no sea así...
Para movernos a otra rama ejecutamos **git checkout {rama}**.

```sh
git checkout remotes/origin/dev #cambiar de rama
git status #muestra la rama actual y los cambios en curso
```

Si en esta rama inspeccionamos el fichero README.md, sorpresa.

---

## 💻 **BANDIT30 → BANDIT31**

Seguimos con Git, tenemos un repositorio de git en _**ssh://bandit30-git@localhost:2220/home/bandit30-git/repo**_, que debemos clonar.

Como en los tres casos anteriores, clonamos usando git clone y la clave ssh del nivel, e inspeccionamos el único fichero que hay (README.md):

```
Just and empty file... muahaha
```

Ahora el fichero no contiene ni siquiera el usuario ni la contraseña "oculta". Como en los casos anteriores, podemos usar **git log** y **git branch -a** para comprobar si en otras ramas o commits anteriores encontramso más información, pero en este caso el historial de commits está vacío, y no tenemos más ramas aparte de la principal (master).

Quizás en esta ocasión, la contraseña no esté en un fichero de texto.

### Git tag (etiquetas)
Existe otro concepto en git, las etiquetas o _tags_. Estas sirven para marcar puntos específicos en el historial de commits. Orgánicamente son punteros que apuntan a un commit determinado.

Algunos comandos interesantes en Tagging:
- **`git tag -l`**: muestra todas las tags creadas en el repositorio.
- **`git tag -a {name} -m {message}`**: crea un nuevo tag con el nombre y el mensaje dados.
- **`git show {tag}`**: Muestra información acerca de un tag específico.

**Solución**
Listamos las etiquetas del repositorio

```bash
bandit30@bandit:~$ git tag -l
secret          #el repositorio tiene un tag con el nombre 'secret'
bandit30@bandit:~$
```

Hemos encontrado un tag con el nombre de 'secret'. Veamos lainformación acerca de esta etiqueta con **git show**:

```bash
git show secret
```

Al hacerlo nos aparece una cadena de texto que parece ser la contraseña al siguiente nivel. Si probamos a loguearnos como bandit31 usando dicha contraseña, tenemos éxito.

---

## 💻 **BANDIT31 → BANDIT32**

Continuamos con Git, tenemos un repositorio de git en _**ssh://bandit31-git@localhost:2220/home/bandit31-git/repo**_, que debemos clonar.

Como en los tres casos anteriores, clonamos usando git clone y la clave ssh del nivel. En esta ocasión, encontramos además del clásico fichero README.md, un fichero **`.gitignore`**. El fichero .gitignore se usa para especificar a git qué ficheros no deben ser tomados en cuenta en el repositorio, por ejemplo, ficheros sensibles con claves de APIs o ficheros de variables de entorno, que no interesa que sean guardados ni enviados al repositorio remoto. La manera de hacerlo suele ser escribiendo los nombres de los ficheros o una expresión regular que represente extensiones o directorios completos.

```bash
#ejemplo de un .gitignore
.env        
.gitignore
*.json      #extensión
/dist       #directorio
```

De momento lo ignoramos, e inspeccionamos el README.md:

```md
This time your task is to push a file to the remote repository

Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master

```

El fichero contiene instrucciones para resolver el nivel. Para obtener la contraseña debemos crear un fichero .txt con el contenido especificado y subirlo al repositorio remoto.

Es importante tener en cuenta la presencia del fichero **.gitignore**, ya que si este contiene la extensión .txt o el nombre del fichero, no podremos subirlo al repositorio remoto.

**Solución**

Primero, nos desplazamos al directorio /repo y creamos el fichero con el contenido:

```bash
cd /repo && echo "May I come in?" > key.txt
```

Para añadir el fichero al repositorio de git empleamos el comando **`git add`**

```
git add key.txt
```

Sin embargo, al ejecutar este comando se nos avisará de que por mucho que se añada el fichero, este no será tenido en cuenta porque aparece como ignorado en el fichero **.gitignore**.

![alt text](image-1.png)

Para resolver esto tenemos dos opciones:
1. Eliminar/cambiar el fichero .gitignore para que deje de ignorar archivos txt.
2. Forzar la ejecución de **git add** con el flag -f, ignorando el .gitignore.

Una vez hecho esto, podemos checkear que ha sido correctamente añadido con el comand **`git status`**:

```bash
bandit31@bandit:~$ git status
On branch master
Your branch is up to date with 'origin/master'
...
...
Changes to commit:
    new file: key.txt
...
...
```

El fichero *key.txt* debería aparecer como nuevo cambio. Guardamos los cambiso creando un nuevo commit:

```bash
git commit -m "level 31"
```

Para subir el nuevo commit al repositorio remoto, empleamos el comando **`git push`**:

```bash
#git push [remoto] [rama]
git push origin master
```

Si lo hemos hecho correctamente, veremos la contraseña del siguiente nivel.

---

## 💻 **BANDIT32 → BANDIT33**

Último nivel de Bandit. En este nivel no hay unas instrucciones específicas, pero al conectarnos por ssh con el usuario bandit33, nos aparece el siguiente mensaje :

```sh
WELCOME TO THE UPPERCASE SHELL
>>
```

La shell que tenemos ahora no nos permite ejecutar ningún comando ya que nos traduce el input del teclado a mayúsculas. Si escribimos el comando ls, lo interpreta como LS.

**Solución**

La variable $0 en Linux contiene el nombre del proceso que se está ejecutando. En nuestro caso el proceso es /bin/bash. $0 no contiene caracteres del abecedario, por lo que puede ser interpretado como es por la UPPERCASE SHELL.

Si escribimos **`$0`** en la terminal, obtendremos un proceso bash común. Si ejecutamos **whoami**, veremos que somos el usuario bandit33, por lo que ya podríamos acceder a la contraseña de este usuario en el directorio _/etc/bandit_pass_.
