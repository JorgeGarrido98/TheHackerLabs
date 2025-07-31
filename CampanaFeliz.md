# 🔔 Resolución del CTF "Campana Feliz" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** CampanaFeliz  
**Dificultad:** Media  
**Enfoque:** Base64, enumeración web, fuerza bruta, escalada con Webmin

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV -Pn 192.168.1.127
```

Puertos abiertos:

- 22 → SSH
- 80 → Apache HTTP
- 8088 → Servicio web sospechoso
- 10000 → Webmin

---

## 🌐 2. Análisis del puerto 8088

El código fuente HTML del puerto 8088 revela un string codificado en base64.

---

## 🔓 3. Decodificación base64

```bash
echo "<cadena>" | base64 -d
```

Se obtiene una pista que sugiere una ruta sensible.

---

## 🧭 4. Enumeración con Gobuster

```bash
gobuster dir -u http://192.168.1.127 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php
```

Ruta interesante detectada: `shell.php`

---

## 💣 5. Exploiting vía shell.php

Se intenta inyección o ejecución remota a través del archivo.

---

## 🔐 6. Fuerza bruta SSH con Hydra

Se utiliza `hydra` para obtener acceso:

```bash
hydra -l webadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.127
```

Credenciales válidas encontradas:  
- Usuario: `webadmin`  
- Contraseña: `********`

---

## 📂 7. Enumeración local

Lectura de `/etc/passwd` revela varios usuarios.

---

## 🧾 8. Fichero webadmin.txt

Archivo revela posible contraseña o credencial adicional.

---

## ⚙️ 9. Acceso al panel Webmin (puerto 10000)

Se ingresa al panel Webmin utilizando credenciales válidas. Desde ahí, se configura una reverse shell o comando con privilegios elevados.

---

## 🧨 10. Reverse Shell y escalada

Se obtiene acceso como usuario privilegiado desde Webmin configurando un payload (revshell).

---

## 🧑‍💻 11. Acceso root

Mediante privilegios desde el panel Webmin o abuso de configuración, se obtiene shell como **root**.

---

## 🏁 12. Flags encontradas

- `flag.txt` en home de usuario
- `root.txt` en `/root/`

---

## ✅ Conclusión

Pasos clave:

1. Análisis y extracción de pista en base64
2. Enumeración con gobuster y explotación de `shell.php`
3. Fuerza bruta SSH → acceso como `webadmin`
4. Acceso a Webmin (puerto 10000)
5. Escalada con reverse shell y obtención de flags

Un reto bien equilibrado que mezcla enumeración, reversing y explotación web realista.

