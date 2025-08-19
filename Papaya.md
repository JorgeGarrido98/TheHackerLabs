# 🟠 Resolución del CTF "Papaya" | TheHackersLabs

**Plataforma:** TheHackersLabs  
**Nombre de la máquina:** Papaya  
**Dificultad:** Media  
**Enfoque:** FTP anónimo, explotación de CMS Elkarte, zip cracking, escalada con `scp`

---

## 🔍 1. Escaneo de puertos

```bash
nmap -p- -- open -sS -sC -sV -- min-rate 5000 -n -vvv -Pn 192.168.1.140 -oN escaneo. txt
```

<img width="526" height="393" alt="1-nmap" src="https://github.com/user-attachments/assets/438911d5-4e05-46f8-8232-b8a9150ea45a" /><br>

<img width="867" height="624" alt="1-nmap2" src="https://github.com/user-attachments/assets/a3de061b-3826-4d52-a993-129beb5fe842" /><br>

Puertos abiertos:

- 21 → FTP (vsftpd)
- 22 → SSH
- 80 → Apache HTTP

---

## 📂 2. Enumeración de FTP

Se permite acceso anónimo. Dentro encuentro un fichero:

```
/secret.txt
```

<img width="372" height="215" alt="2-ftp-anonymous-secret txt" src="https://github.com/user-attachments/assets/da1a459b-2b46-40f4-bd3a-d88aca587fd6" /><br>

Lo descargo y contiene texto cifrado.

<img width="256" height="80" alt="3-cat-secret txt" src="https://github.com/user-attachments/assets/4004ecdc-0dc8-40b4-828c-45f160acf2aa" />

---

## 🔓 3. Descifrado de mensaje

El contenido de `secret.txt` se descifra (parece cifrado ROT13) y revela un mensaje que parece no ser de utilidad jajaja:

<img width="290" height="24" alt="4-decrypt-secreto txt" src="https://github.com/user-attachments/assets/1efb6fdb-e0d7-40b4-b8b0-e0929b8d538d" />

---

## 🌐 4. Acceso a CMS Elkarte

Accedo a la web en el puerto 80. Es un CMS llamado **Elkarte**.

Probando un rato doy con credenciales por defecto y entro al panel de admin:

> admin:password

La funcionalidad de instalación de temas es vulnerable a la ejecución de código PHP:

<img width="1126" height="627" alt="6-elkarte-theme-install" src="https://github.com/user-attachments/assets/4b1ed036-5d9f-4325-b45e-16e6f5937f3e" />

---

## 🧨 5. Reverse Shell vía ZIP modificado

Uso `Reverse Shell Generator` -> PHP Pentest Monkey:

<img width="547" height="477" alt="7-revshell-generator" src="https://github.com/user-attachments/assets/d06ce3b4-424b-4570-a28e-e0154ff024fa" /><br>

Lo guardo en un archivo llamado `revshell.php` y lo comprimo en un .zip:

<img width="295" height="62" alt="8-revshell zip" src="https://github.com/user-attachments/assets/3da4b7df-2178-4679-b8ea-eaf2e043cb10" /><br>

Instalo la reverse shell:

<img width="298" height="82" alt="9-install-revshell" src="https://github.com/user-attachments/assets/46e1bf9b-2359-4f2f-a0f0-d598ec9663af" /><br>

Me pongo en escucha con Netcat por el puerto 4444 y ejecuto la reverse shell:

<img width="164" height="20" alt="10-ejecuto-revshell" src="https://github.com/user-attachments/assets/34ca7b2b-cdab-4468-af4f-725595f51954" /><br>

<img width="494" height="117" alt="11-acceso-wwwdata" src="https://github.com/user-attachments/assets/0d6bc25d-7f17-48f3-beb6-e8d490febb8f" /><br>

> Resultado: acceso como **www-data**

---

## 🔐 6. Extracción de credenciales

Desde el archivo `settings.php` de Elkarte se extraen credenciales de la base de datos pero en la base de datos no hay nada interesante:

<img width="140" height="171" alt="13-var-www-html-elkarte-settings php-password-db" src="https://github.com/user-attachments/assets/08b4c4d8-7ceb-457c-a1b5-e07b53995de9" />

---

## 🗂️ 7. Descubrimiento de ZIP protegido

En `/opt/` se encuentra `pass.zip`, protegido con contraseña.

<img width="428" height="547" alt="14-encontramos-opt-pass zip" src="https://github.com/user-attachments/assets/95e1b2d9-13eb-4513-8216-50d669cf3795" /><br>

<img width="366" height="176" alt="14-opt7pass zip" src="https://github.com/user-attachments/assets/60c7b033-402b-4ef0-ab39-a9133dee2a0a" /><br>

1. Se crackea con `john` + `rockyou.txt`.
2. Se extrae un fichero que contiene la contraseña de un usuario del sistema.

<img width="496" height="139" alt="15-john-crack-pass zip" src="https://github.com/user-attachments/assets/04974330-2318-4388-b2ae-03d268fdb19f" /><br>

<img width="388" height="118" alt="16-unzip-pass zip" src="https://github.com/user-attachments/assets/03bb354d-1fcb-4787-88d5-ace65ff656e3" /><br>

🔑 Credenciales obtenidas:

> User -> papaya <br>
> Password -> papayarica

---

## 🧠 8. Escalada de privilegios

```bash
sudo -l
```

<img width="417" height="79" alt="18-sudo -l" src="https://github.com/user-attachments/assets/a5550ea6-2928-4e3f-9048-a41407ea218a" /><br>

Se encuentra que `papaya` puede ejecutar `scp` como root sin contraseña.

---

## 🚀 9. Escalada con SCP (GTFOBins)

Usando la técnica de GTFOBins para `scp`:

<img width="422" height="115" alt="19-gtfobins-scp" src="https://github.com/user-attachments/assets/0a8ce868-21fc-4640-8566-19d766e0a6ff" /><br>

```bash
sudo scp -S /bin/bash x y:
```

<img width="240" height="73" alt="20-escalada-root" src="https://github.com/user-attachments/assets/90b5890d-4c19-48d8-a5c1-424ef5b372c4" /><br>

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

