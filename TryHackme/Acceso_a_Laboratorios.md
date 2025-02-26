## Acceso a los Laboratorios
Para conectarnos a la plataforma, nos conectaremos mediante VPN. Nos descargaremos el archivo .ovpn que nos facilita la plataforma y nos podemos conectar con el siguiente comando 

```bash
sudo openvpn <archivo_vpn.ovpn>
```
*Si utilizamos Kali Linux, recordar que la interfaz de red es tun0. Podemos comprobar nuestra IP con alguno de los siguientes comandos 
```bash
ifconfig <interfad_red>
``` 
```bash
ip a s tun0
```
La IP de la máquina objetivo, nos la dará la propia plataforma de TryHackme una vez iniciemos la máquina, por lo que no tendremos que escanear la red en busca de equipos. Fácil y al pie. 
