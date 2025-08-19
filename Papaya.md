# 🟠 Resolución del CTF "Papaya" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Papaya  
**Dificultad:** Media  
**Enfoque:** FTP anónimo, explotación de CMS Elkarte, zip cracking, escalada con `scp`

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV -Pn 192.168.1.127
```

Puertos abiertos:

- 21 → FTP (vsftpd)
- 22 → SSH
- 80 → Apache HTTP

---

## 📂 2. Enumeración de FTP

Se permite acceso anónimo. Dentro encontramos un fichero:

```
/secret/secret.txt
```

Lo descargamos y contiene texto cifrado.

---

## 🔓 3. Descifrado de mensaje

El contenido de `secret.txt` se descifra (parece base64 con cifrado ROT13 o similar) y revela un mensaje que sugiere mirar la web o una credencial parcial.

---

## 🌐 4. Acceso a CMS Elkarte

Accedemos a la web en el puerto 80. Es un CMS llamado **Elkarte**.  
La funcionalidad de instalación de temas es vulnerable a la ejecución de código PHP.

---

## 🧨 5. Reverse Shell vía ZIP modificado

1. Se genera una reverse shell en PHP.
2. Se empaqueta como un "theme" de Elkarte.
3. Se sube a través del panel de administración.
4. Se ejecuta accediendo al archivo `.php`.

Resultado: acceso como **www-data**

---

## 🔐 6. Extracción de credenciales

Desde el archivo `settings.php` de Elkarte se extraen credenciales de la base de datos, una de ellas se reutiliza más adelante.

---

## 🗂️ 7. Descubrimiento de ZIP protegido

En `/opt/` se encuentra `pass.zip`, protegido con contraseña.

1. Se crackea con `john` + `rockyou.txt`.
2. Se extrae un fichero que contiene la contraseña de un usuario del sistema.

---

## 🔑 8. Acceso como usuario papaya

```bash
su papaya
# Contraseña: <extraída del ZIP>
```

---

## 🧠 9. Escalada de privilegios

```bash
sudo -l
```

Se encuentra que `papaya` puede ejecutar `scp` como root sin contraseña.

---

## 🚀 10. Escalada con SCP (GTFOBins)

Usando la técnica de GTFOBins para `scp`:

```bash
sudo scp -S /bin/bash x y:
```

Esto proporciona una shell como **root** 🏁

---

## ✅ Conclusión

Pasos clave:

1. Acceso inicial por FTP anónimo
2. Análisis y explotación de CMS Elkarte
3. Reverse shell como www-data
4. Cracking de ZIP para obtener credenciales
5. Escalada a root usando `scp`

Un CTF excelente para practicar múltiples técnicas: enumeración, explotación de CMS, cracking y privilege escalation.

