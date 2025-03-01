# 🔐 Bandit: niveles 20-33

## 💻 **BANDIT19 → BANDIT20**

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

## 💻 **BANDIT20 → BANDIT21**

En este nivel aprenderemos a utilizar la herramienta **netcat** en modo LISTEN, para conseguir la password que habilita el siguiente nivel debemos emplear el binario setuid **`suconnect`** que se encuentra en el directorio. Este programa realiza lo siguiente: establece una conexión a localhost en un puerto dado. 

Es importante entender que este programa actuará como **_cliente_**, por lo que nosotros deberemos crear la conexión que escuche, es decir la parte del **_servidor_**. Esto se puede lograr con la herramienta netcat de Linux y la opción -l.

**Solución**

En una terminal:
```sh
nc -lnvp 9000
```
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

Un ejemplo de fichero **_crontab_** en un sistema puede ser el siguiente. En este fichero, se espeficia el programa/script, y a qué intervalos debe crontab ejecutar dicho programa/script. 

Un asterisco en una de las columnas de tiempo indica básicamente "todo". Cinco asteriscos supondría que se ejecuta cada minuto:

```bash
# Minuto  Hora  Día del Mes  Mes  Día Semana  Comando
0 3 * * * /usr/bin/backup.sh     # Backup diario a las 3 AM
0 0 1 * * /usr/bin/monthly_report.sh  # Genera un informe el primer día de cada mes a medianoche
```

En este nivel en lugar de usar un fichero **crontab** emplean varios ficheros dentro del directorio **/etc/cron.d**. Listando dicho directorio encontramos un fichero llamado cronjob_bandit22, el nombre implica que nos interesa.

**Solución:**

El contenido del fichero de cron es el siguiente. Nos muestra que hay un .sh que se ejecuta "silenciosamente" cada minuto:

```bash
0 3 * * * /usr/bin/cronjob_bandit22.sh &> /dev/null 
```

> **NOTA:** *&> /dev/null* indica que no se muestre la salida proporcionada en terminal

Si hacemos un less al contenido de ese fichero veremos que lo que hace el .sh es copiar la contraeña que necesitamos a otro fichero al cual si tenemos permisos para leer:

```sh
#!/bin/bash
chmod 644 /tmp/xxxx
cat /etc/bandit_pass/bandit22 > /tmp/xxx
```
Si se inspecciona el fichero /temp/xxx se obtiene la contraseña. 