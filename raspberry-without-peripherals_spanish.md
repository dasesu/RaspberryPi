# Conectarse a una Raspberry Pi sin perifericos.

Se asume una raspberry con interfaz de red inalámbrica, y una microsd con una distro raspbian (`2020-02-13-raspbian-buster`) aunque esto funciona también en otras distros de raspbian.
   
Insertamos la microsd en nuestro sistema operativo para realizar algunos cambios previos, observamos dos particiones `boot` y `rootfs`, dentro de la partición `boot` y crearmos un archivo llamandolo `ssh`, y agregamos el valor "1" como contenido. Esto habilitará el protocolo ssh para poder acceder remotamente cuando se cargue el Sistema Operativo.

```bash
$ cd /media/myuser/boot/
$ sudo touch ssh
$ echo "1" > ssh
```

Dentro de la partición `rootfs`, editamos el archivo `wpa_supplicant.conf` que se encuentra en del directorio ``/etc/wpa_supplicant`

```bash
$ cd /media/myuser/rootfs/etc/wpa_supplicant
$ sudo vi wpa_supplicant.conf
```

Agregamos la configuración de la red a la que nos vamos a conectar, para que se conecte automaticamente al iniciar el Sistema Operativo.

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=ES

network={
   ssid="NETWORK-NAME"
   psk="NWTWORK-PASSWORD"
   key_mgmt=WPA-PSK
}
```

Extraemos la microsd del computador y la insertamos en la raspberry pi, conectamos la raspberry al puerto de entrada de energia para que inicie el sistema operativo, como no tenemos monitor esperamos hasta que haya terminado de cargar, aproximadamente 2 minutos.

Para poder conectarnos a la raspberry tenemos que encontrar que dirección ha tomado. Hay varias formas de hacer esto, podemos, por ejemplo acceder a la configuración de nuestro router y buscar la raspberry entre los dispositivos conectados. En redes con pocos dispositivos conectados, podemos tratar de encontrar la dirección IP manualmente hasta encontrar la correcta. Nosotros usaremos el comando `nmap`, cambiando la dirección de red por la que tenga nuestra red, podemos además filtrar el rango de busqueda, por ejemplo desde la dirección  192.168.0.100 a la dirección 192.168.0.255.

```bash
sudo nmap 192.168.0.100-255
```

Entre el resultado obtenido, podemos ver algo parecido a lo siguiente, lo que nos indica que la raspberry se encuentra en la dirección `192.168.0.101`

```
Nmap scan report for 192.168.0.101
Host is up (0.019s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
MAC Address: B8:27:EB:FF:3A:EC (Raspberry Pi Foundation)
```

Nos conectamos usando ssh a la raspberry desde nuestro pc, usando el usuario `pi` y contraseña `raspberry` que son los valores que trae por defecto.

```bash 
$ ssh pi@192.168.0.101
```

Ya conectados a la raspberry podemos realizar el resto de configuraciones con el comando:
```
sudo raspi-config
```
