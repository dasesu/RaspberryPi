# Creating an automated keyboard with a Raspberry pi

Algunos modelos de Raspberry como la Pi Zero y Pi Zero W cuentan con un puerto
USB OTG, esto permite que sean configuradas para comportarse como dispositivos
perifericos virtuales como por ejemplo teclados, camaras, impresoras, entre
otros, además, como versiones recientes de raspbian incluyen ConfigFS una
raspberry puede comportarse como dispositivo USB de multiple proposito

## Modulos soportados:
**Serial** (g_serial)   
**Ethernet** (g_ether)   
**Mass storage** (g_mass_storage)   
**MIDI** (g_midi)   
**Audio** (g_audio)   
**Keyboard/Mouse** (g_hid)   
**Mass storage and Serial** (g_acm_ms)   
**Ethernet and Serial** (g_cdc)   
**Multi** (g_multi) – Allows you to configure 2 from Ethernet, Mass storage and Serial   
**Webcam** (g_webcam)   
**Printer** (g_printer)   
**Gadget tester** (g_zero)   


## Configurar Raspberry como teclado usb.
Agregamos la instrucción `dtoverlay=dwc2` al final del archivo /boot/config.txt y las lineas `dwc2` y `libcomposite` al archivo /etc/modules

```bash 
echo "dtoverlay=dwc2" | sudo tee -a /boot/config.txt
echo "dwc2" | sudo tee -a /etc/modules
sudo echo "libcomposite" | sudo tee -a /etc/modules
```

ConfigFS será cargado en /sys/kernel/config/usb_gadget luego de reiniciar la raspberry.

La manera de interactuar con ConfigFS es por medio del arbol de directorios, para definir un nuevo dispositivo en ConfigFS se debe crear un directorio en `/sys/kernel/config/usb_gadget`, esto puede hacerse manualmente pero los cambios se perderían al iniciar el sistema, asi que necesitamos que se haga automaticamente cada vez que se inicie el sistema operativo, para eso creamos un archivo `/usr/bin/usbkeyboardconf` y lo hacemos ejecutable

```bash
sudo touch /usr/bin/usbkeyboardconf
sudo chmod +x /usr/bin/usbkeyboardconf
sudo vi /usr/bin/usbkeyboardconf
```

El contenido que debemos agregar al archivo `usbkeyboardconf` recien creado se muestra a continuación
```bash
#!/bin/bash
cd /sys/kernel/config/usb_gadget/
mkdir -p myKeyboard
cd myKeyboard

echo 0x1d6b > idVendor # Linux Foundation
echo 0x0104 > idProduct # Multifunction Composite Gadget
echo 0x0100 > bcdDevice # v1.0.0
echo 0x0200 > bcdUSB # USB2

mkdir -p strings/0x409
echo "ABCDEF" > strings/0x409/manufacturer
echo "ABCDEF Keyboard" > strings/0x409/product
echo "0123456789" > strings/0x409/serialnumber

mkdir -p configs/c.1/strings/0x409
echo "Some Configuration" > configs/c.1/strings/0x409/configuration
echo 200 > configs/c.1/MaxPower

# Add functions here
mkdir -p functions/hid.usb0
echo 1 > functions/hid.usb0/protocol
echo 1 > functions/hid.usb0/subclass
echo 8 > functions/hid.usb0/report_length
echo -ne \\x05\\x01\\x09\\x06\\xa1\\x01\\x05\\x07\\x19\\xe0\\x29\\xe7\\x15\\x00\\x25\\x01\\x75\\x01\\x95\\x08\\x81\\x02\\x95\\x01\\x75\\x08\\x81\\x03\\x95\\x05\\x75\\x01\\x05\\x08\\x19\\x01\\x29\\x05\\x91\\x02\\x95\\x01\\x75\\x03\\x91\\x03\\x95\\x06\\x75\\x08\\x15\\x00\\x25\\x65\\x05\\x07\\x19\\x00\\x29\\x65\\x81\\x00\\xc0 > functions/hid.usb0/report_desc

ln -s functions/hid.usb0 configs/c.1/

# End functions
ls /sys/class/udc > UDC
```

Editamos `/etc/rc.local` para que la configuración se ejecute al iniciar el SO agregando lo siguiente antes de exit:  
```bash
...
/usr/bin/usbkeyboardconf
exit 0
```

Ahora solo debemos configurar lo que se va a escribir automaticamente, para eso crearemos un script en python, podemos llamarlo como queramos, en este ejemplo lo llamamos `keyboard_example.py`.  
```python
#!/usr/bin/env python3

NULL_CHAR = chr(0)

def write_report(report):
    with open('/dev/hidg0', 'rb+') as fd:
        fd.write(report.encode())


# H (press shift and H)
write_report(chr(32)+NULL_CHAR+chr(11)+NULL_CHAR*5)

# e
write_report(NULL_CHAR*2+chr(8)+NULL_CHAR*5)

# ll
write_report(NULL_CHAR*2+chr(15)+NULL_CHAR*5)
write_report(NULL_CHAR*8)
write_report(NULL_CHAR*2+chr(15)+NULL_CHAR*5)

# o
write_report(NULL_CHAR*2+chr(18)+NULL_CHAR*5)

# SPACE
write_report(NULL_CHAR*2+chr(44)+NULL_CHAR*5)

# W (press shift and W)
write_report(chr(32)+NULL_CHAR+chr(26)+NULL_CHAR*5)

# o
write_report(NULL_CHAR*2+chr(18)+NULL_CHAR*5)

# r
write_report(NULL_CHAR*2+chr(21)+NULL_CHAR*5)

# l
write_report(NULL_CHAR*2+chr(15)+NULL_CHAR*5)

# d
write_report(NULL_CHAR*2+chr(7)+NULL_CHAR*5)

# ! (press shift and 1)
write_report(chr(32)+NULL_CHAR+chr(30)+NULL_CHAR*5)

# Release all keys
write_report(NULL_CHAR*8)
```

Editamos nuevamente el archivo  `/etc/rc.local` para llamar a nuestro script
```bash
...
/usr/bin/usbkeyboardconf
sleep 10
/home/pi/keyboard_example.py
exit 0
```

Para probarlo, la raspberry debe conectarse directamente al PC por el puerto de datos con un cable que soporte OTG, no a el puerto de power.

Al conectar la raspberry a un PC ( por el puerto de datos unicamente ), se cargará el sistema operativo, y se ejecutará el archivo `keyboard_example.py` que tecleará automaticamente  `Hello World!`.

**Refference:**
* [https://gist.github.com/gbaman/50b6cca61dd1c3f88f41](https://gist.github.com/gbaman/50b6cca61dd1c3f88f41)
* [https://www.rmedgar.com/blog/using-rpi-zero-as-keyboard-setup-and-device-definition](https://www.rmedgar.com/blog/using-rpi-zero-as-keyboard-setup-and-device-definition)
* [https://cdn-learn.adafruit.com/downloads/pdf/turning-your-raspberry-pi-zero-into-a-usb-gadget.pdf](https://cdn-learn.adafruit.com/downloads/pdf/turning-your-raspberry-pi-zero-into-a-usb-gadget.pdf)
* [https://www.isticktoit.net/?p=1383](https://www.isticktoit.net/?p=1383)
* [https://gpiozero.readthedocs.io/en/stable/pi_zero_otg.html](https://gpiozero.readthedocs.io/en/stable/pi_zero_otg.html)
