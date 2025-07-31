# 🔔 Resolución del CTF "Campana Feliz" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** CampanaFeliz  
**Dificultad:** Media  
**Enfoque:** Base64, enumeración web, fuerza bruta, escalada con Webmin

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -sS -sC -sV -Pn 192.168.1.127
```

<img width="554" height="637" alt="1-nmap" src="https://github.com/user-attachments/assets/4c8b2fcd-ccb1-4659-8df4-7a905131d585" /><br>

Puertos abiertos:

- 22 → SSH
- 8088 → Servicio web sospechoso
- 10000 → Webmin

---

## 🌐 2. Análisis del puerto 8088

El código fuente HTML del puerto 8088 revela un string codificado en base64.

<img width="518" height="123" alt="2-puerto8088-codigofuente" src="https://github.com/user-attachments/assets/ff8a0fe6-a594-4c87-a9f6-4e936632e311" />

---

## 🔓 3. Decodificación base64

```bash
echo "<cadena>" | base64 -d
```

<img width="548" height="108" alt="3-decode-base64" src="https://github.com/user-attachments/assets/e809277d-8f15-4e43-8879-61586d585e51" /><br>

Se obtiene una pista que sugiere un usuario `campana`.

---

## 🧭 4. Enumeración con Gobuster

```bash
gobuster dir -u http://192.168.1.127 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php
```

<img width="508" height="368" alt="4-gobuster" src="https://github.com/user-attachments/assets/0af25ea7-3928-4cef-a3e1-154eebd84335" /><br>

Ruta interesante detectada: `shell.php`

<img width="206" height="152" alt="5-shell php" src="https://github.com/user-attachments/assets/e2ba4615-fa3d-448a-9b6c-f60ac426b341" />

---

## 🔐 5. Fuerza bruta SSH con Hydra

Se utiliza `hydra` para obtener acceso:

```bash
hydra -l webadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.127
```

<img width="1041" height="113" alt="6-hydra" src="https://github.com/user-attachments/assets/cf16c574-d822-4c1d-8e2f-796595b41bc4" /><br>

Credenciales válidas encontradas:  
- Usuario: `campana`  
- Contraseña: `lovely`

---

## 📂 6. Enumeración local

Lectura de `/etc/passwd` revela varios usuarios.

<img width="298" height="303" alt="7-etc-passwd" src="https://github.com/user-attachments/assets/12b6758e-a525-4d90-a9e3-af3d43e49794" /><br>

> Intento hydra con el usuario `bob` pero no da resultado.

---

## 🧾 7. Fichero webadmin.txt

Archivo en `/opt` revela posible contraseña o credencial adicional:

```bash
cat /opt/CMS\ Webmin.txt
```

<img width="214" height="205" alt="8-webadmin txt" src="https://github.com/user-attachments/assets/e3956671-dcbc-4026-aa21-7477caca9a24" /><br>

🔗 URL -> http://IP:10000<br>
🔑 Credenciales obtenidas: santaclaus:FelizNavidad2024

---

## ⚙️ 8. Acceso al panel Webmin (puerto 10000)

Se ingresa al panel Webmin utilizando credenciales válidas:

<img width="177" height="181" alt="10-puerto10000" src="https://github.com/user-attachments/assets/593931f9-9466-4290-bd45-a295ff389256" /><br>


Desde ahí, se configura una reverse shell:

<img width="1127" height="311" alt="11-revshell" src="https://github.com/user-attachments/assets/84cd4fb5-9776-4ce4-a5fe-cb32db29233f" /><br>
<img width="326" height="76" alt="12-acceso-root" src="https://github.com/user-attachments/assets/f560806b-8e69-4d81-9da6-220399c8b256" />

---

## 🧑‍💻 10. Acceso como root y flags

Mediante privilegios desde el panel Webmin o abuso de configuración, se obtiene shell como **root** y se leen las flags:

<img width="278" height="119" alt="13-flag-user" src="https://github.com/user-attachments/assets/14f33f45-23b9-4374-979b-31c772b432d6" /><br>
<img width="382" height="192" alt="14-flag-root" src="https://github.com/user-attachments/assets/ed86b7e0-a480-46e1-bbb5-c58ea6ada3fd" />

---

## ✅ Conclusión

Pasos clave:

1. Análisis y extracción de pista en base64
2. Enumeración con gobuster y explotación de `shell.php`
3. Fuerza bruta SSH → acceso como `webadmin`
4. Acceso a Webmin (puerto 10000)
5. Escalada con reverse shell y obtención de flags

Un reto bien equilibrado que mezcla enumeración, reversing y explotación web realista.

