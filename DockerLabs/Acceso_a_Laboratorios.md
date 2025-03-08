Nos aseguramos de tener Docker instalado en el equipo; si no está instalado, ejecutamos el siguiente comando

```bash
sudo apt install docker.io
```
Descomprimimos el archivo .zip 

```bash
7z x [Archivo.zip]
```
Tendremos dos archivos. trust.tar (que es la propia máquina) y un script auto_deploy.sh. Simplemente ejeuctaremos el script para desplegar el laboratorio. 

```bash
sudo bash auto_deploy.sh trust.tar
```

Una vez terminado el laboratorio, con CTRL + C, para eliminar todo del sistema. 



