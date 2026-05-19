
# Servidor de Ficheros con Samba (UD10)

**Alumno:** Santiago Hernandez  
**Número de lista:** 10  

---

## Descripción

Configuración de un servidor de ficheros en Linux (Zorin) utilizando Samba, permitiendo el acceso desde un cliente Windows 11 con distintos niveles de permisos según usuario y recurso compartido.

---

## Entorno

- Sistema servidor: Zorin OS (Linux)
- Sistema cliente: Windows 11
- Red: NAT
- IP servidor: 10.0.2.15  
- IP cliente: 10.0.2.28  

---

## Instalación

```
sudo apt update
sudo apt install samba -y
````

***

## Estructura de directorios

```
/srv/samba/publica
/srv/samba/compartida
```

Permisos configurados:

```
sudo chown root:sambashare /srv/samba/*
sudo chmod 770 /srv/samba/*
```

***

## Usuarios

Creación de usuarios sin login:

```
sudo useradd -m -s /sbin/nologin -G sambashare samba1
sudo useradd -m -s /sbin/nologin -G sambashare samba2
sudo useradd -m -s /sbin/nologin -G sambashare samba3
```

Alta en Samba:

```
sudo smbpasswd -a samba1
sudo smbpasswd -a samba2
sudo smbpasswd -a samba3
```

***

## Configuración de Samba

Archivo: `/etc/samba/smb.conf`

### Configuración global

```
[global]
   workgroup = WORKGROUP
   netbios name = SAMBA10
   security = user
```

***

### Carpeta pública

```
[publica]
   path = /srv/samba/publica
   browsable = yes
   read only = yes
   valid users = samba1 samba2 samba3
```

***

### Carpetas personales

```
[homes]
   browseable = no
   read only = no
   valid users = %S
```

***

### Carpeta compartida

```
[compartida]
   path = /srv/samba/compartida
   browsable = yes
   read only = yes
   valid users = samba1 samba2
   write list = samba1
   force group = sambashare
   veto files = /*.zip/
```

***

## Ajuste de permisos reales

Para evitar problemas de escritura:

```
sudo chown -R root:sambashare /srv/samba/compartida
sudo chmod -R 770 /srv/samba/compartida
sudo chmod g+s /srv/samba/compartida
```

***

## Acceso desde Windows

Rutas de acceso:

```
\\10.0.2.15\publica
\\10.0.2.15\samba1
\\10.0.2.15\compartida
```

***

## Comportamiento esperado

* **publica**
  * Lectura: permitida
  * Escritura: denegada

* **homes**
  * Acceso solo al directorio propio
  * Lectura y escritura permitidas

* **compartida**
  * samba1: lectura y escritura
  * samba2: solo lectura
  * samba3: sin acceso

* Archivos `.zip` no visibles desde clientes

***

## Notas

* Windows 11 bloquea acceso anónimo, por lo que se ha usado autenticación por usuario.
* Es necesario configurar correctamente permisos en Linux además de Samba.
* Se utiliza el bit SGID para mantener coherencia en los permisos de grupo.
