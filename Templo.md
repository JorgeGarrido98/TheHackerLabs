# 🛕 Resolución del CTF "Templo" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Templo  
**Dificultad:** Media/Alta  
**Enfoque:** LFI, bypass de validación, ROT13, cracking de ZIP, escalada con LXD

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV -Pn 192.168.1.127
```

**Puertos abiertos:**
- 22 → SSH
- 80 → Apache HTTP (página: "Wow")
- 9090 → Apache HTTP (página: "NAMARI")

---

## 🧭 2. Enumeración web

### En el puerto 80 (WOW):
Se encuentra un `clue.txt` que menciona otra dirección (`http://NAMARI:9090/`).

### En el puerto 9090 (NAMARI):
Hay una web vulnerable que permite subir archivos y muestra directorios públicos. Se encuentra funcionalidad vulnerable al listado de ficheros y a **LFI** (`../../../../etc/passwd`).

---

## 🛠️ 3. Revisión de código fuente

Accediendo al código fuente de `upload.php`, se descubre que el nombre de los archivos se codifica mediante **ROT13**.  
También se aplica ROT13 al nombre de un fichero PHP en la ruta index.

---

## 🧨 4. Reverse shell y ejecución

1. Se prepara una reverse shell en PHP.
2. Se cifra con ROT13 para que el nombre del archivo pase desapercibido al filtrado.
3. Se sube con éxito a través del formulario.
4. Se ejecuta y se obtiene acceso inicial como **www-data**.

---

## 📦 5. Escalada de privilegios – ZIP protegido

En `/opt/` se encuentra `backup.zip` que solicita contraseña.

- Se crackea con `john` y `rockyou.txt`.
- Contraseña encontrada: `********`
- Contiene credenciales de `rodgar`.

---

## 🔑 6. Acceso como usuario rodgar

```bash
su rodgar
# Contraseña: <extraída del zip>
```

---

## 📈 7. Enumeración con linpeas

LinPEAS muestra que el sistema tiene instalado **LXD versión 5.21.3 LTS**.

---

## 🚀 8. Escalada a root con LXD

Se utiliza la técnica clásica de escalada con contenedor:

1. Importar una imagen Alpine (`alpine/3.18`) en local.
2. Crear contenedor con volumen montado a `/root` del sistema.
3. Obtener shell root al acceder desde el contenedor al host.

---

## 🏁 9. Conclusión

- 🗂️ LFI con bypass por codificación
- 🧠 ROT13 como técnica para ocultar nombre de archivos
- 🔐 Crackeo de ZIP protegido
- ⚙️ Escalada con LXD

Un CTF muy completo, ideal para practicar técnicas combinadas de explotación web y post-explotación en Linux.

