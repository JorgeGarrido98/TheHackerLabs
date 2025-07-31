# 🏃 Resolución del CTF "Runners" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Runners  
**Dificultad:** Media/Alta  
**Enfoque:** SQLi, exfiltración de archivos, cracking, uso de pspy, escalada con Docker

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.1.127
```

<img width="917" height="611" alt="1-nmap" src="https://github.com/user-attachments/assets/d49dc112-b575-47a1-b4fe-6523175d9100" /><br>

Se detectan:

- Puerto 22 → OpenSSH
- Puerto 80 → Apache con sitio web activo
- Puerto 2222 → OpenSSH

---

## 🌐 2. Enumeración web

Con `gobuster` se encuentra `/post.php?id=1`, lo que sugiere posible SQLi:

```bash
gobuster dir -u http://192.168.1.127 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

<img width="520" height="333" alt="2-gobuster" src="https://github.com/user-attachments/assets/b19a956c-9a96-413c-aab3-0538593b01ee" />

---

## 💥 3. Explotación SQLi con sqlmap

<img width="1126" height="373" alt="3-post phpid=1" src="https://github.com/user-attachments/assets/c03706dd-d43a-4df6-a0ae-c42c8ddfb9b0" /><br>

Confirmamos vulnerabilidad:

```bash
sqlmap -u "http://192.168.1.127/post.php?id=1" --dbs
```
<img width="642" height="275" alt="4-sqlmap-dbs" src="https://github.com/user-attachments/assets/2898bc38-81dd-4984-8b0b-610e52863319" /><br>
<img width="600" height="458" alt="5-qslmap-tables" src="https://github.com/user-attachments/assets/560a0a13-52bb-4b32-9a2a-4ece523990df" /><br>
<img width="661" height="189" alt="6-sqlmap-users" src="https://github.com/user-attachments/assets/4f8b449a-c96b-44d3-b772-3fe70ede7bd3" /><br>

Se descubre la base de datos `blog`, y dentro de ella la tabla `users` con usuarios y hashes.

---

## 🔓 4. Cracking de hash

Extraemos el hash y lo crackeamos:

<img width="922" height="260" alt="7-crackeo-password" src="https://github.com/user-attachments/assets/a09abed1-1f85-449e-8a24-46c223aaa6e0" /><br>

🔑 Contraseña obtenida: `runner`

---

## 💻 5. Acceso SSH como usuario `david` en el Puerto 2222

```bash
ssh david@192.168.1.127
# Contraseña: runner
```

<img width="469" height="191" alt="8-ssh-login" src="https://github.com/user-attachments/assets/7448742c-2331-4ed4-9a9d-a5140b4352fe" /><br>

Se obtiene acceso como usuario `david`.

---

## 📂 6. Revisión del sistema

Explorando el sistema encontramos `/home/david/.hidden/credenciales.zip`.

<img width="355" height="174" alt="9-credenciales php" src="https://github.com/user-attachments/assets/ca884693-a8df-451f-a159-cc61a5dc027b" /><br>

Lo copiamos con `scp`:

<img width="546" height="144" alt="10-scp-credenciales php" src="https://github.com/user-attachments/assets/2e41e9d9-a28e-4907-971a-293020e551e6" /><br>

Extraemos el hash:

<img width="548" height="62" alt="11-hash-credenciales php" src="https://github.com/user-attachments/assets/7a8af067-e605-45d3-85c3-3ad2526d265e" /><br>

Crackeamos la contraseña con `john`:

<img width="479" height="90" alt="12-john-credenciales php" src="https://github.com/user-attachments/assets/087a5545-b951-464d-934e-8459dea09bb0" /><br>

Con la contraseña obtenida `rockandroll` conseguimos extraer un fichero `.xlsx` con más credenciales.:

<img width="561" height="188" alt="14-leemos-credenciales xlsx" src="https://github.com/user-attachments/assets/0f3cbad3-721e-4bbb-b705-2f649e4194d2" />

🔑 Credenciales obtenidas: `maria:4br53#j6p78mq#zbvc`

---

## 📈 7. Enumeración con pspy en usuario `maria`

Se observa que un script `backup.sh` se ejecuta periódicamente como root (`maria` tiene permisos de escritura).

<img width="1123" height="557" alt="15-pspy" src="https://github.com/user-attachments/assets/93220348-9972-447b-a61b-0aec56065930" /><br>

Se modifica el script para inyectar una reverse shell.

<img width="545" height="341" alt="16-revshell-backup sh" src="https://github.com/user-attachments/assets/4e6b4897-ca55-4a92-abb0-ebf36ef8a94d" />

---

## 🚪 9. Acceso root y leemos fichero TODO_LIST.txt

Escalada a root que ejecuta el script:

<img width="407" height="65" alt="17-escalada-root-p2222" src="https://github.com/user-attachments/assets/25226961-27db-4f00-b61c-ceee601a86d4" /><br>

Leemos en la carpeta /root el fichero `TODO_LIST.txt`:

<img width="546" height="184" alt="18-credenciales-ian" src="https://github.com/user-attachments/assets/ff1ef193-ef69-4811-a10a-8344b7ce99b8" /><br>

🔑 Credenciales obtenidas: `ian:iambatman`

---

## 💻 10. Acceso SSH como usuario `ian` en el Puerto 22

Leo la flag de usuario:

<img width="353" height="133" alt="19-flag-user" src="https://github.com/user-attachments/assets/ddf301e8-53a3-4bdf-9343-a961e420d175" /><br>

En `/home/elliot` encuentro un fichero llamado `miscredenciales.psafe3`:

<img width="546" height="176" alt="20-miscredenciales psafe3" src="https://github.com/user-attachments/assets/abfbe87d-f051-4c7c-b46c-89d56ca2326f" /><br>

Extraemos el hash y crackeamos la contraseña de acceso con `john`:

<img width="475" height="128" alt="21-john-miscredenciales psafe3" src="https://github.com/user-attachments/assets/cd164637-c0cc-4469-a5dc-ff5248567773" /><br>

Entro con `Password Safe`:

<img width="237" height="172" alt="22-elliot-password" src="https://github.com/user-attachments/assets/1048fc26-969e-40ed-b925-6ca0d4fcb1f3" /><br>

🔑 Credenciales obtenidas: `elliot:HwbE80ZOtZQdkYB`

---

## 🛠️ 11. Escalada a root vía Docker

Con `id` y búsqueda de binarios SUID, se identifica que el usuario pertenece al grupo `docker`.

<img width="398" height="58" alt="25-id-docker" src="https://github.com/user-attachments/assets/5b22d40f-18c1-4313-aab1-151779ec5218" /><br>

Se lanza un contenedor Docker interactivo montando el sistema y accediendo como root:

```bash
docker run -v /:/mnt --rm -it ubuntu chroot /mnt
```

<img width="431" height="95" alt="26-escalada-root-docker-p22" src="https://github.com/user-attachments/assets/8f0ba114-2ef8-420c-b9a6-d53e073fa040" />

---

## 🏁 12. Flag final

Una vez dentro del contenedor, accedemos a `/root/root.txt` y conseguimos la flag.

<img width="181" height="35" alt="27-flag-root" src="https://github.com/user-attachments/assets/61210026-7357-4cd7-abec-9cd45c7a8c05" />

---

## ✅ Conclusión

Pasos clave:

1. SQLi → enumeración de base de datos
2. Cracking de hashes → acceso SSH
3. Revisión de ficheros y extracción de credenciales
4. Uso de pspy → escalada mediante script vulnerable
5. Acceso como root vía Docker

Un reto técnico y realista, ideal para poner a prueba tus habilidades de post-explotación.

