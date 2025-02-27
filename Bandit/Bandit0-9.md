

# 🔐 Bandit: niveles 0 - 9

Para comenzar a practicar desde el nivel 0, conectar por ssh con el usuario bandit0 a la dirección bandit.labs.overthewire.org en el puerto 2220.

```
ssh -p 2220 bandit0@bandit.labs.overthewire.org
```

## 💻 **BANDIT0 → BANDIT1**

El nivel más básico. La contraseña se encuentra en el mismo directorio del usuario **bandit0**, concretamente en:

```
/home/bandit0/readme
```

### 📄 Comandos para mostrar el contenido de ficheros:
- **`cat`** *(concatenar)*: muestra el contenido de un fichero en la consola.
- **`more` / `less`**: muestran el contenido de un fichero, útiles para ficheros largos.
- **`vi` / `vim`, `pico` / `nano`**: editores de texto con diversas funcionalidades.

---

## 💻 **BANDIT1 → BANDIT2**

El concepto es similar al nivel anterior, pero ahora el archivo se llama **`-`**, lo que puede generar conflictos.  
La solución es especificar la ruta completa (relativa o absoluta):

> **Solución:**
>
> ```sh
> cat /home/bandit1/-
> ```

---

## 💻 **BANDIT2 → BANDIT3**

En este nivel, el fichero de contraseñas se llama **`spaces in this filename`**, es decir, contiene espacios (carácter *blankspace*, ASCII 32).

### ⚙️ Formas de escapar los espacios:
- **Usando el carácter de escape `\`:**

  ```sh
  cat spaces\ in\ this\ filename
  ```

- **Usando comillas simples o dobles:**

  ```sh
  cat "spaces in this filename"
  ```

---

## 💻 **BANDIT3 → BANDIT4**

La contraseña se encuentra ahora en el subdirectorio **`inhere`** dentro de la carpeta:

```
/home/bandit3
```

### 📁 Comandos útiles:
- **`cd`**: Permite navegar entre directorios.
  - `cd ..` te lleva al directorio padre.
- **`ls`**: Lista el contenido del directorio.
  - `ls -lah` muestra **todos** los ficheros (incluso los ocultos, que comienzan con `.`).

> **Solución:**
>
> ```sh
> cat inhere/...hidden_from_you
> ```

---

## 💻 **BANDIT4 → BANDIT5**

Aquí existen varios archivos, pero solo uno contiene la contraseña. Buscar uno a uno sería muy ineficiente.

### Carácter `*`:
El carácter asterisco `*` sirve como comodín en expresiones regulares, y puede usarse para casar con cualquier carácter. 
Dado que todos tienen un nombre de archivo que cumple el mismo patrón *(-file0{algo})*, podemos sustituir {algo} por * y usar el comando **more** para mostrar todos los archivos cuyo nombre case con -file0*.

> **Nota:** Una vez más, cuidado con el nombre de los ficheros ya que empiezan todos por '-'.

> **Solución:**
>
> ```sh
> more inhere/-file0*
> ```

---

## 💻 **BANDIT5 → BANDIT6**

El fichero se encuentra dentro de un directorio en `/inhere` y se indica que ocupa **1033 bytes**.

### 🔍 Uso del comando `find`:
`find` permite buscar ficheros según condiciones específicas (nombre, tamaño, usuario, fecha de modificación, etc.), similar a una consulta SQL.

- La opción **`-size 1033c`** busca ficheros de **1033 bytes**.
- 	La opción **`-exec`** dentro de find es muy interesante y permite ejecutar uno o varios comandos adicional para cada fichero encontrado. Su uso es -exec [comando] '{}' ';'. De esta forma, podemos ejecutar cat para cada archivo encontrado y mostrar el archivo al mismo tiempo que lo buscamos.


> **Solución:**
>
> ```sh
> find . -size 1033c -exec cat '{}' \;
> ```

  > **Nota:** Conviene mirar las páginas del manual de find con el comando man find para este nivel.

---

## 💻 **BANDIT6 → BANDIT7**

El siguiente archivo se encuentra en alguna parte del sistema de ficheros del servidor. Sabemos lo siguiente:
- El fichero ocupa **33 bytes**.
- Es propiedad de **bandit7** pero puede ser leído por **bandit6**.  
  Usa `groups` para verificar que tu usuario pertenezca al grupo **bandit6**.

Podemos volver a usar el comando find junto con las opciones de las páginas del manual para buscar por los criterios establecidos. Como se va a intentar acceder a muchos ficheros y algunos nuestro usuario no tiene permiso para leerlos, nos van a aparecer múltiples errores de permiso en la terminal. 
Esto puede evitarse redirigiendo la salida de standart error a null con **`2>/dev/null.`** El 2 hace referencia al descriptor de fichero de stderr (stdout, la salida estándar, es 1). 
Con el uso de exec además podemos hacer que se muestre el fichero en pantalla cuando se encuentre, como en el nivel anterior.

> **Solución:**
>
> ```sh
> find / -size 33c -user bandit7 -group bandit6 -exec cat {} ';' 2>/dev/null
> ```

---

## 💻 **BANDIT7 → BANDIT8**

La contraseña se encuentra en el fichero **`data.txt`**, el cual es muy grande y contiene muchas líneas. Sabemos que la contraseña está junto a la palabra **`millionth`**.

### 🔍 Uso de `grep`:
`grep` filtra el contenido de un fichero usando expresiones regulares y muestra las líneas donde hay coincidencia.

> **Soluciones:**
>
> ```sh
> cat data.txt | grep millionth
> ```
>
> o de forma directa:
>
> ```sh
> grep millionth data.txt
> ```

---

## 💻 **BANDIT8 → BANDIT9**

En este nivel, la contraseña se encuentra en la única línea **no repetida** del fichero **`data.txt`**, mientras que todas las demás líneas se repiten.

### ⚙️ Combinando `sort` y `uniq`:
- **`sort`** ordena las líneas para que las repetidas queden juntas.
- **`uniq -u`** muestra solo las líneas que no se repiten.

> **Solución:**
>
> ```sh
> cat data.txt | sort | uniq -u
> ```

> **Consejo adicional:** si quieremos un poco más de info. como cuántas veces se repite cada línea podemos usar sort, luego uniq -c para contar, sort -n para ordenar por nº de repes y head -n5 para mostrar las 5 primeras líneas de la salida.
>
> ```sh
> cat data.txt | sort | uniq -c | sort -n | head -n5
> ```

