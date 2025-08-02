# 🧩 Resolución del CTF "CryptoLabyrinth" | The HackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** CryptoLabyrinth  
**Dificultad:** Media  
**Enfoque:** Enumeración web, cracking de hashes, análisis de criptografía, fuerza bruta, escalada con sudo

---

## 🔍 1. Escaneo inicial con Nmap

```bash
nmap -p- -sS -sC -sV --min-rate 2000 -Pn 192.168.1.127
```

Puertos abiertos:

- 22 → OpenSSH
- 80 → Apache HTTP Server

---

## 🌐 2. Enumeración Web

Con `gobuster` se detectan rutas ocultas. Se accede a `/hidden/`, donde hay ficheros para descargar.

---

## 💾 3. Análisis de ficheros

Se encuentran ficheros de Bob y Alice. En el de Bob hay varios hashes etiquetados (`bob-hash1`, `hash2`, ...).  
Se identifican tipos de hash y se preparan para crackeo.

---

## 🔓 4. Cracking de hashes

Con `hashcat` y uso de diccionarios, se logra obtener varias contraseñas de Bob.  
Con una de ellas se puede leer parcialmente el archivo cifrado de Alice.

---

## 🧠 5. Fuerza bruta de clave parcial

En la lectura del archivo cifrado de Alice aparece una posible clave parcial: `2LWxmDsW0**`  
Se genera una wordlist combinando letras y números para completar los dos caracteres faltantes.

---

## 💥 6. Fuerza bruta SSH

Con `hydra` se prueba la wordlist generada contra el usuario `alice`:

```bash
hydra -l alice -P wordlist.txt ssh://192.168.1.127
```

✅ Se encuentra la contraseña correcta.

---

## 💻 7. Acceso como Alice

Se accede por SSH y se ejecuta `sudo -l` para verificar privilegios. Alice puede ejecutar ciertos binarios como root.

---

## 📦 8. Análisis de binarios disponibles

En la home de Alice hay un archivo con un string que sugiere otra contraseña potencial.  
Se intenta crackear también con `hydra` la contraseña de `root` usando esta pista.

---

## 🔓 9. Acceso como root

Se accede con éxito como root tras crackear la contraseña de superusuario.

---

## 🏁 10. Flag final

Se accede a `/root/root.txt` y se recupera la flag.

---

## ✅ Conclusión

1. Enumeración web y análisis de ficheros cifrados
2. Cracking de múltiples hashes
3. Wordlist generada para clave incompleta
4. Fuerza bruta SSH contra Alice
5. Escalada a root mediante sudo o contraseña crackeada

Un reto que combina esteganografía criptográfica, cracking y fuerza bruta de forma progresiva. Ideal para afianzar técnicas de criptografía y post-explotación.

