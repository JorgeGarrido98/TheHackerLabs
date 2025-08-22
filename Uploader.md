# 📤 Resolución del CTF "Uploader" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Uploader  
**Dificultad:** Fácil  
**Enfoque:** Upload de reverse shell, cracking de ZIP, escalada con `/tar`

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -- open -sS -sC -sV -- min-rate 5000 -n -vVV -Pn 192. 168.1.116 -oN escaneo. txt
```
<img width="490" height="578" alt="1-nmap" src="https://github.com/user-attachments/assets/da2c8912-b252-43ff-9299-aca6a2fd8ec8" /><br>

Servicios detectados:

- 80 → Apache HTTP Server

---

## 🌐 2. Enumeración Web

Con `gobuster` se identifican directorios importantes y una funcionalidad de subida de archivos.

<img width="532" height="206" alt="2-gobuster" src="https://github.com/user-attachments/assets/44c441b9-d841-4d5d-a51b-a701351a6a58" />

---

## 🧨 3. Explotación vía subida de archivos

1. Se genera una reverse shell PHP.
2. Se sube con éxito a través del formulario (sin filtro de extensión estricto).
3. Se accede a la ruta de subida y se activa la reverse shell.

<img width="548" height="630" alt="3-revshell php" src="https://github.com/user-attachments/assets/0a9f6165-63f1-4f6e-8c17-490e8c149559" /><br>

<img width="821" height="364" alt="4-upload-revshell php" src="https://github.com/user-attachments/assets/12e1b307-bb00-4a65-a150-0217a23da6ac" /><br>

<img width="347" height="185" alt="5-revshell-cargado" src="https://github.com/user-attachments/assets/7d9d2a06-e69a-4a3f-87eb-210646f77fe0" /><br>


Resultado: shell como `www-data`:

<img width="553" height="127" alt="6-conexión-inicial" src="https://github.com/user-attachments/assets/c384a4aa-e859-467a-b8cc-bffa57ab8b18" /><br>

---

## 🧑‍💻 4. Enumeración post-explotación

Se encuentra al usuario `operatorx` en el sistema:

<img width="332" height="101" alt="7-user-operatorx" src="https://github.com/user-attachments/assets/f48cce86-8bb5-46a2-bba3-72901631b0f5" /><br>

En el home se encuentra un archivo `README.txt` que menciona un archivo ZIP:

<img width="328" height="109" alt="8-readme txt" src="https://github.com/user-attachments/assets/b892c350-8514-402a-851b-cc122fc9e186" /><br>

---

## 📦 5. Descubrimiento y análisis del ZIP

Se localiza `file.zip`, lo descargo a mi Kali pero está protegido con contraseña:

<img width="1119" height="119" alt="9-encontramos-file zip-descargamos" src="https://github.com/user-attachments/assets/6d4f49e8-ca41-4de4-bed3-9ca23cec3d74" /><br>

<img width="447" height="355" alt="10-extraer-zip-pide-password" src="https://github.com/user-attachments/assets/8bbaa740-acdd-4b2a-b55b-82b7d2025ce8" /><br>

Analizamos el zip:

```bash
zipinfo -v file.zip
```
<img width="425" height="506" alt="11-zipinfo-credential-notencrypted" src="https://github.com/user-attachments/assets/974a5cae-b673-405e-ae10-6bddbaf9888b" /><br>

<img width="407" height="370" alt="11-zipinfo-credentials txt-encrypted" src="https://github.com/user-attachments/assets/4628866c-17b0-4b2d-80c4-8d35049b4903" /><br>

Se confirma que uno de los archivos está cifrado (`credentials.txt`).

---

## 🔓 6. Cracking del ZIP con John

Se extrae el hash y se crackea con `john` y la wordlist `rockyou.txt`.

<img width="495" height="150" alt="12-cracking-zip-john" src="https://github.com/user-attachments/assets/2152504a-47a2-4098-b45e-0347207d348f" /><br>

Contraseña recuperada → permite descomprimir `credentials.txt`.

---

## 🧠 7. Decodificación de credenciales

El archivo `credentials.txt` contiene una cadena codificada (base64 o similar):

<img width="377" height="364" alt="13-credentials txt" src="https://github.com/user-attachments/assets/6e108dae-6a3a-4822-9963-f5ae7f921a67" /><br>

Se decodifica y revela la contraseña de `operatorx`:

<img width="529" height="191" alt="14-decode-password" src="https://github.com/user-attachments/assets/0345a44a-6ee9-4a84-80e6-e566134deac8" />

---

## 🔑 8. Acceso como `operatorx`

```bash
su operatorx
# Contraseña: <decodificada>
```

<img width="275" height="43" alt="15-login-operatorx" src="https://github.com/user-attachments/assets/937c4f7b-17d4-4659-8ecf-08634b963fb4" />

---

## 🧨 9. Escalada de privilegios

```bash
sudo -l
```

<img width="470" height="76" alt="16-sudo -l" src="https://github.com/user-attachments/assets/995877e5-9466-42fa-ba83-7508dcc3d8c8" /><br>

> Se permite ejecutar `/usr/bin/tar` como root sin contraseña.

Usamos GTFOBins:

```bash
sudo tar -cf /dev/null /dev/null -- checkpoint=1 -- checkpoint-action=exec=/bin/sh
```

<img width="417" height="92" alt="17-gtfobins" src="https://github.com/user-attachments/assets/68bae73e-e5b5-4b15-a0fc-4ecf064e0214" /><br>

<img width="1037" height="104" alt="18-escalada-root" src="https://github.com/user-attachments/assets/98ef1233-5d21-4997-bc1a-5f5c88273ce3" /><br>

Obtenemos shell como **root** 🏁

---

## ✅ Conclusión

Pasos clave:

1. Reverse shell vía subida de archivo PHP
2. ZIP protegido con credenciales → crack con `john`
3. Decodificación de base64 para obtener contraseña
4. Escalada con `tar` usando permisos sudo

Excelente CTF para practicar subida de shells, cracking básico y privilege escalation con binarios comunes.

