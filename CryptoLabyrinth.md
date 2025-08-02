# 🧩 Resolución del CTF "CryptoLabyrinth" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** CryptoLabyrinth  
**Dificultad:** Media  
**Enfoque:** Enumeración web, cracking de hashes, análisis de criptografía, fuerza bruta, escalada con sudo

---

## 🔍 1. Escaneo inicial con Nmap

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -vvv -Pn 172.20.10.3 -oN escaneo.txt
```

<img width="1118" height="534" alt="1-nmap" src="https://github.com/user-attachments/assets/d453ca5a-1660-454a-89b5-6d2edbc1566e" /><br>

Puertos abiertos:

- 22 → OpenSSH
- 80 → Apache HTTP Server

---

## 🌐 2. Enumeración Web

Con `gobuster` se detectan rutas ocultas. Se accede a `/hidden/`, donde hay ficheros para descargar.

<img width="470" height="361" alt="2-gobuster" src="https://github.com/user-attachments/assets/a2567364-85c3-4179-bd5e-e267fd19bbd9" />

---

## 💾 3. Análisis de ficheros

Se encuentran ficheros de Bob y Alice. En el de Bob hay varios hashes etiquetados (`bob-hash1`, `hash2`, ...).  
Se identifican tipos de hash y se preparan para crackeo.

---

## 🔓 4. Cracking de hashes

Con `hashcat` y uso de diccionarios, se logra obtener varias contraseñas de Bob.  

<img width="287" height="338" alt="3-hidden" src="https://github.com/user-attachments/assets/74712af8-db3c-4a4e-94d6-af0149f7e721" /><br>
<img width="242" height="249" alt="5-bob-hashes" src="https://github.com/user-attachments/assets/e50e19e8-6ec5-4719-b6aa-858b4ae4d099" /><br>
<img width="517" height="147" alt="6-bob-hash1" src="https://github.com/user-attachments/assets/65f9f4ac-020b-486e-9e9e-7730a166ff67" /><br>
<img width="505" height="137" alt="6-bob-hash2" src="https://github.com/user-attachments/assets/73115d2a-c923-448a-8a90-4c2b2556d6f1" /><br>
<img width="503" height="140" alt="6-bob-hash3" src="https://github.com/user-attachments/assets/e8f34397-56f9-4138-a509-c9e019201bcf" /><br>
<img width="506" height="140" alt="6-bob-hash4" src="https://github.com/user-attachments/assets/1ba6024f-cc62-487c-99b0-917dc6d2e68a" /><br>
<img width="752" height="585" alt="7-hashcat" src="https://github.com/user-attachments/assets/d541f11a-7c10-4256-8969-496b976d10b1" /><br>
<img width="227" height="24" alt="7-bob-hash5" src="https://github.com/user-attachments/assets/771addaf-6192-4ebd-8570-5da941cea92e" />

---

## 🧠 5. Fuerza bruta de clave parcial

En el código fuente de la plantilla de Apache aparece una posible clave parcial en un comentario: `2LWxmDsW0**`

<img width="173" height="97" alt="image" src="https://github.com/user-attachments/assets/680aabf1-7c8c-44bf-8619-ab874529ca92" /><br>

Se genera una wordlist combinando letras y números para completar los dos caracteres faltantes:

<img width="224" height="304" alt="9-wordlist-passwords-2LWxmDsW0" src="https://github.com/user-attachments/assets/d2ff0a2e-6b05-455e-b7e7-b131cbca72c8" /><br>

---

## 💥 6. Fuerza bruta SSH

Con `hydra` se prueba la wordlist generada contra los usuarios `bob` y `alice`:

```bash
hydra -L users.txt -P posibles_passwords.txt -u ssh://172.20.10.3 -f -V
```

<img width="573" height="218" alt="10-hydra" src="https://github.com/user-attachments/assets/8c893da7-f9ca-48cc-811a-0863a2fb58c4" /><br>
<img width="515" height="265" alt="11-hydra-password-encontrada" src="https://github.com/user-attachments/assets/d32983b7-2210-4f22-a02e-eea1ed76569f" /><br>

✅ Se encuentra la contraseña correcta.

---

## 💻 7. Acceso como Bob

Se accede por SSH y se ejecuta `sudo -l` para verificar privilegios. Bob puede ejecutar ciertos binarios para convertirse en Alice con `/env`:

<img width="583" height="71" alt="12-sudo -l" src="https://github.com/user-attachments/assets/5f3db652-04b4-482a-8ac6-01ac3601f535" /><br>
<img width="385" height="44" alt="13-login-alice" src="https://github.com/user-attachments/assets/6236ac34-c72f-418a-8e14-179b7c9daa69" />

---

## 📦 8. Análisis de fichero `.secreto.txt`

En `/mnt` Alice creó un archivo con un string que sugiere otra contraseña potencial.  

<img width="311" height="176" alt="14-posible-password" src="https://github.com/user-attachments/assets/049450fb-c791-4b25-883a-cde2d0bba24f" />

---

## 🔓 9. Crackeo contraseña root

Se intenta crackear también con `hydra` la contraseña de `root` o `debian`, que son los dos usuarios que no tenemos acceso, usando esta pista:

<img width="1040" height="219" alt="16-hydra-root-debian" src="https://github.com/user-attachments/assets/ef096407-9b6b-4644-a914-40f403785cba" /><br>
<img width="520" height="173" alt="17-hydra-password-encontrada-root" src="https://github.com/user-attachments/assets/8933e8c3-885d-44ec-a1ea-63d98d77aeff" /><br>

✅ Se encuentra la contraseña correcta.

---

## 🏁 10. Accedemos como root y flag final

Accedemos como root con su password:

<img width="254" height="33" alt="18-escalada-root" src="https://github.com/user-attachments/assets/3da0cb61-4a5a-475b-a9ae-43f566579d9a" /><br>

Se accede a `/root/root.txt` y se recupera la flag:

<img width="286" height="148" alt="19-flag-root" src="https://github.com/user-attachments/assets/4655a740-72e7-4973-81c8-ab07f975853f" />

---

## ✅ Conclusión

1. Enumeración web y análisis de ficheros cifrados
2. Cracking de múltiples hashes
3. Wordlist generada para clave incompleta
4. Fuerza bruta SSH contra Alice
5. Escalada a root mediante sudo o contraseña crackeada

Un reto que combina esteganografía criptográfica, cracking y fuerza bruta de forma progresiva. Ideal para afianzar técnicas de criptografía y post-explotación.

