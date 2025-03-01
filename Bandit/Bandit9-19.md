# 🔐 Bandit: niveles 9-19

## 💻 **BANDIT9 → BANDIT10**

En este nivel, el fichero **`data.txt`** no es un fichero común de texto, sino que se encuentra en un formato ilegible.  
El problema nos dice que la conraseña se encuentra en una de las pocas cadenas de texto legibles, junto a algunos caracteres _'='_ (ojo, nos vale para usar **grep**).

El comando **`strings`** permite extraer las cadenas de texto legibles de un fichero. combinando el uso de strings con grep podemos filtrar el contenido del fichero y encontrar la contraseña.

**Solución:**
```sh
cat data.txt | grep -e '=='
```

---

## 💻 **BANDIT10 → BANDIT11**
El fichero **`data.txt`** (que contiene la contraseña para el siguiente lvl) ahora se encuentra cifrado en base64.

La herramienta base64 de Linux permite cifrar/descifrar cadenas de texto y archivos en este formato. Simplemente, usando este comando con la opción -d (decode) podemos descifrar
el fichero y obtener la contraseña.

**Solución:**
```sh
base64 -d data.txt
```
---

## 💻 **BANDIT11→ BANDIT12**
En esta ocasión, el fichero data.txt ha sido cifrado empleando un método de cifrado muy popular denominado cifrado del César.
En concreto nos dicen que cada letra ha sido rotada 13 posiciones dentro del alfabeto (la clave es 13).

### 🔐 Cifrado del César: 
Se trata de un método de cifrado ancestral y muy antiguo, el cual consiste en rotar las letras del mensaje 'x' posiciones en el alfabeto (x sería la clave de cifrado). 

1. **Primero**, se establece una nueración para cada letra del alfabeto, por ejemplo, 

> A : 0, B : 1, c : 2 ... Z : 25, a : 26 , b : 27, ... z : 51. 


2. **Después**, para cifrar se suma el valor de la clave a cada letra del texto, por ejemplo, para un texto claro Hola, y una clave x = 5:


```rust
Hola ---> 7 40 37 26  + 5 = 12 45 42 31  ---> Ktsf
```

Se debe tener en cuenta que las operaciones de suma son mediante aritmética modular con módulo 52, es decir, si al rotar las 'x' posiciones nos pasamos de 51, volvemos a contar desde 0.

El comando **`tr (translate)`** permite rotar 'x' posiciones cada letra. Conociendo el número de posiciones (13) podemos establecer mediante una expresión regular. Primero, se le ha de pasar la expresión regular para definir qué parte del texto debe ser rotado. Para este caso, estableceremos la expresión 'A-Za-z', es decir, todas las letras del abecedario. Segundo, se le ha de pasar una cadena que represente por qué letras se deben sustituir. 13 posiciones supone sustituir A-M,B-N....

**Solución:**

```sh
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```
---

## 💻 **BANDIT12 → BANDIT13**
En esta ocasión, el fichero **`data.txt`** ha sido comprimido múltiples veces. De primeras, el fichero está en forma de "hexdump" (hexadecimal). En esta forma, se puede distinguir el formato del fichero observando la "firma de formato".

Si visualizamos el fichero con **`cat`**, vemos un montón de líneas. Realmente lo que se está mostrando es el contenido del fichero en forma hexadecimal, con ocho bytes por línea. Los primeros bytes que se muestran se corresponden con la firma de formato. Podemos consultar en internet (https://en.wikipedia.org/wiki/List_of_file_signatures) para ver qué tipo de fichero essegún su firma. Tras una rápida búsqueda obtenemos.

La solución a este ejercicio es verificar mediante file o haciendo un hexdump el tipo de archivo comprimido que es, y usar las diferentes herramientas que te pongo debajo para ir descomprimiendo archivos sucesivas veces hasta que aparezca lel fichero de contraseña.

Cuando al hacer file [fichero] veas:

```vbnet
fichero: ASCII text
```
Habrás llegado a tu destino -.-

#### Comandos útiles: - 
- **`file`**: le pasamos un fichero y nos dice de qué tipo es 
- **`xdd`**: hace un hexdump de un archivo, o lo deshace 
- **`unzip`**: descomprime archivos .zip 
- **`gzip`**: comrpime/descomprime en formato gzip 
- **`bzip2`**: otra herramienta para comprimir/descomprimir

<br>

> Nota: cuidado con la extensión de los ficheros. Aunque file te diga que es un tipo de fichero específico, si no le colocas su debida extensión (puedes renombrar el fichero con **mv**), puede que la herramienta no sea capaz de descomprimir el archivo.

---

## 💻 **BANDIT13 → BANDIT14**
Este nivel es bastante sencillo si sabes cómo funciona **`SSH`**. Una de las alternativas a usar usuario y contraseña al acceder mediante SSH a un sistema es emplear un algoritmo criptográfico de par de claves como RSA. 

Investigando en las páginas del manual de SSH podrás observar que el parámetro **-i** te permite adjuntar un fichero de clave privada para loguearte en un sistema.

### localhost y 127.0.0.1 🔄

El nombre **`localhost`** hace referencia a la máquina actual, al igual que la dirección IP 127.0.0.1 (dirección IP loopback) la cual es reservada para este propósito. Podemos emplear tanto el nobre 'localhost' como la dirección IP loopback para establecer conexiones de red con nuestra propia máquina. En este nivel, esa conexión debe ser establecida mediante SSH.

Teniendo la clave privada de **bandit14** y suponiendo que el fichero de clave pública se encuentra en el servidor en el directorio del usuario bandit14, podemos loguearnos en una nueva shell en la propia máquina mediante ssh y ganar acceso como bandit14.

**Solución:**

sh
Copiar
Editar
ssh -p 2220 -i ssh_private_key bandit14@localhost

---

## 💻 **BANDIT14 → BANDIT15**
Para resolver este nivel debemos de alguna forma enviar la contraseña de nuestro usuario actual (bandit14) al puerto 30000 de la misma máquina.

En esta máquina, todas las contraseñas se encuentran en el directorio **/etc/bandit_pass/{usuario}**, y cada usuario solamente puede leer su propia contraseña. Por tanto, para leer la contraseña del nivel actual (14) deberemos introducir o enviar de alguna manera esta esta contraseña por un socket hacia nuestra propia máquina. Cuando hablamos de _socket_, nos referimos a estaqblecer una conexión a nivel TCP/UDP mediante una IP y puerto, se trata de un concepto abstracto.

### Herramienta nc (netcat) 🐱: 
Esta herramienta permite abrir sockets TCP/UDP tanto en modo LISTEN (recibir) como para enviar datos. Combinando esta herramienta con alguna de las vistas para mostrar texto en la terminal mediante el operador pipe (|) podemos resolver este nivel y obtener la contraseña para el siguiente.

**Solución:**

```sh
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```
---

## 💻 **BANDIT15 → BANDIT16**
En este nivel la idea es exactamente la misma que en nivel anterior, pero con dos diferencias: 
- El puerto al que se debe realizar la conexión mediante socket y enviar la contraseña es el **30001**.
- Este puerto funciona a través de TLS 1.3 y emplea cifrado junto con un certificado de servidor, por lo que la conexión debe realizarse de forma diferente.

### 🔐 OpenSSL: 

OpenSSL es una suite de herramientas relacionadas con el cifrado de datos y la expedición de certificados digitales, entre otras cosas. La **`herramienta s_client`** de OpenSSL implementa un cliente SSL/TLS permite depurar puertos SSL/TLS, por lo que podemos realizar la conexión al puerto mediante esta herramienta usando la opción -connect. 

La conexión que se establece es interactiva de alguna manera y todas las teclas pulsadas son enviadas por el socket, por lo que para tener éxito en esta ocasión **no usaremos el operador de tubería** para redirigir la salida de cat, si no que estableceremos primero la conexión con s_client y luego pegaremos o escribiremos la contraseña.

> Nota: Cuando hablamos de SSL y TLS, realmente son lo mismo. TLS (Transport Layer Security) vendría a ser la versión moderna de SSL (Socket Security Layer).

**Solución:**

```sh
openssl s_client -connect localhost:30001
```
Una vez ejecutado el comando, enviar la contraseña a través de la terminal para recibir la del siguiente nivel.

---

## 💻 **BANDIT16 → BANDIT17**

Para obtener la contraseña al siguiente nivel, se debe enviar la contraseña del nivel actual a un servicio de la máquina que corre en uno de los puertos entre el **31000 y el 32000**. Solamente uno de los puertos corre el servicio capaz de enviar la contraseña para el siguiente nivel. El resto, simplemente repetirá loo que se le mande (hará eco).

### 👁️ NMap (Network mapper)
**`Nmap`** es una herramienta potentísima y muy útil para hacer escaneo en redes. Permite descubrir los puertos que está utilizando una máquina y qué servicios corren en esos puertos, entre otras cosas. La opción **-p** permite especificar tanto puertos específicos como rangos de puertos a escanear. De esta forma también reducimos el tiempo que toma nmap para hacer el escaneo.

Una opción interesante de nmap es la **detección de versiones (opción -sV)**. Usando esta opción podremos distinguir los puertos que usan SSL/TLS de los que no lo hacen.

```sh
 nmap -p31000-32000 -sV localhost
```

Cuando termine el escaneo, veremos en pantalla algo como lo siguiente: 

![Captura de pantalla 2025-03-01 121104](https://github.com/user-attachments/assets/b8667d4b-7990-4974-8621-222cf156cfa5)

Vemos que nmap ha sido capaz de reconcer 5 puertos abiertos en el rango dado. 2 de ellos emplean **ssl**, son los que nos interesan. uno de ellos corre el servicio **echo**, es decir, repetirá todo lo que le enviemos (no nos interesa🦜). El puerto al cual deberíamos probar es pues el **`31790`**.  

**Solución:**

Establecemos la conexión con **s_client** a ese puerto, igual que en el nivel anterior. Una vez conectados, introducimos la contraseña. Si lo hemos hecho bien, el servidor nos devolverá una clave RSA privada que, si hemos completado el nivel 13 → 14, sabremos usar.

```sh
openssl s_client -connect localhost:31790 -ign_eof
```

>La opción _-ign_eof_ evita conflictos con el carácter de return a la hora de introducir la contraseña.

## 💻 **BANDIT17 → BANDIT18**

en este nivel tenemos dos ficheros en la carpeta personal, **passwords.old y passwords.new**. La contraseña de este nivel se encuentra en el fichero **passwords.new**, en la única línea en la cual difieren ambos ficheros.

El comando **`diff`** permite extraer diferencias entre ficheros en Linux. Podemos usarloo para extraer la línea concreta en la cual difieren ambos ficheros y obtener rápidamente la contraseña sin tener que buscar a ojo.

**Solución:**

```sh
diff passwords.old passwords.new
```
Tammbién se pueden usar los comandos sort y uniq como se vio en niveles anteriores para sacar la línea exclusiva.

## 💻 **BANDIT18 → BANDIT19**

En este nivel la contraseña se encuentra en un fichero llamado **readme**. El problema aquí es que alguien ha modificado el fichero **.bashrc**. Este fichero se ejecuta cada vez que el usuario se loguea, y alguien ha añadido la orden **`exit 0`** para que el sistema nos expulse nada más loguearnos.

Sin embargo aunque no podemos usar la terminal con ssh, si que podemos ejecutar comandos a través de ssh como cat, more, less o ls. Si se tiene en cuenta esto la solución es muy simple.

**Solución:**

Existen múltiples soluciones. La más sencilla, hacer cat o less a través de ssh para ver el fichero readme remoto.
```sh
ssh -p 2220 bandit18@bandit.labs.overthewire.org cat readme
```
También podemos ejecutar una terminal tipo **sh**, que es má sencilla y no emplea el fichero .bashrc por lo que no nos expulsará.
```sh
ssh -p 2220 bandit18@bandit.labs.overthewire.org -t sh
```
O podemos intentar descargarnos el fichero readme a nuestra máquina atacante mediante scp, utilidad que funciona a través de ssh.

```bash
scp -P 2220 bandit18@bandit.labs.overthewire.org:/home/bandit18/readme .
```