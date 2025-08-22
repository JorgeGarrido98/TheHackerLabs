# 📤 Resolución del CTF "Uploader" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Uploader  
**Dificultad:** Media  
**Enfoque:** Upload de reverse shell, cracking de ZIP, escalada con `python3`

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV -Pn 192.168.1.127
```

Servicios detectados:

- 22 → SSH
- 80 → Apache HTTP Server

---

## 🌐 2. Enumeración Web

Con `gobuster` se identifican directorios importantes y una funcionalidad de subida de archivos.

---

## 🧨 3. Explotación vía subida de archivos

1. Se genera una reverse shell PHP.
2. Se sube con éxito a través del formulario (sin filtro de extensión estricto).
3. Se accede a la ruta de subida y se activa la reverse shell.

Resultado: shell como `www-data`.

---

## 🧑‍💻 4. Enumeración post-explotación

Se encuentra al usuario `operatorx` en el sistema.

En el home se encuentra un archivo `README.txt` que menciona un archivo ZIP.

---

## 📦 5. Descubrimiento y análisis del ZIP

Se localiza `file.zip` pero está protegido con contraseña. Usando:

```bash
zipinfo -v file.zip
```

Se confirma que uno de los archivos está cifrado (`credentials.txt`).

---

## 🔓 6. Cracking del ZIP con John

Se extrae el hash y se crackea con `john` y la wordlist `rockyou.txt`.

Contraseña recuperada → permite descomprimir `credentials.txt`.

---

## 🧠 7. Decodificación de credenciales

El archivo `credentials.txt` contiene una cadena codificada (base64 o similar).  
Se decodifica y revela la contraseña de `operatorx`.

---

## 🔑 8. Acceso como `operatorx`

```bash
su operatorx
# Contraseña: <decodificada>
```

---

## 🧨 9. Escalada de privilegios

```bash
sudo -l
```

Se permite ejecutar `/usr/bin/python3` como root sin contraseña.

Usamos GTFOBins:

```bash
sudo python3 -c 'import os; os.system("/bin/sh")'
```

Obtenemos shell como **root** 🏁

---

## ✅ Conclusión

Pasos clave:

1. Reverse shell vía subida de archivo PHP
2. ZIP protegido con credenciales → crack con `john`
3. Decodificación de base64 para obtener contraseña
4. Escalada con `python3` usando permisos sudo

Excelente CTF para practicar subida de shells, cracking básico y privilege escalation con binarios comunes.

