# 🧪 Laboratorio 2: LDAP y Kerberos

## Información General

- **Dificultad**: Intermedia
- **Duración estimada**: 75 minutos
- **Prerrequisitos**: 
  - Laboratorio 1 completado
  - Conocimientos básicos de Linux
  - Comprensión de conceptos de autenticación

## Objetivos

Al completar este laboratorio, serás capaz de:

1. Instalar y configurar un servidor OpenLDAP
2. Crear y gestionar usuarios en LDAP
3. Comprender la estructura jerárquica de LDAP
4. Configurar un Key Distribution Center (KDC) de Kerberos
5. Autenticar usuarios usando tickets de Kerberos
6. Integrar LDAP con Kerberos

---

## Parte 1: LDAP y autenticación básica (45 min)

### Objetivo

Comprender el funcionamiento de un servicio de directorio y la autenticación básica de usuarios.

### Paso 1.1: Instalación de OpenLDAP

```bash
# Actualizar repositorios
sudo apt update

# Instalar OpenLDAP y utilidades
sudo apt install slapd ldap-utils -y

# Reconfigurar slapd para configuración inicial
sudo dpkg-reconfigure slapd
```

Durante la reconfiguración:
- Omitir configuración de OpenLDAP: **No**
- Nombre de dominio DNS: `ejemplo.com`
- Nombre de organización: `Ejemplo Corp`
- Contraseña del administrador: (elegir una segura)
- Base de datos: **MDB**
- ¿Eliminar base de datos al purgar?: **No**
- ¿Mover base de datos antigua?: **Sí**

### Paso 1.2: Verificar la instalación

```bash
# Verificar que el servicio está corriendo
sudo systemctl status slapd

# Verificar la estructura básica
ldapsearch -x -b "dc=ejemplo,dc=com"
```

### Paso 1.3: Crear estructura de unidades organizacionales

Crear archivo `base_estructura.ldif`:

```ldif
dn: ou=Usuarios,dc=ejemplo,dc=com
objectClass: organizationalUnit
ou: Usuarios

dn: ou=Grupos,dc=ejemplo,dc=com
objectClass: organizationalUnit
ou: Grupos

dn: ou=Docentes,dc=ejemplo,dc=com
objectClass: organizationalUnit
ou: Docentes

dn: ou=Alumnos,dc=ejemplo,dc=com
objectClass: organizationalUnit
ou: Alumnos
```

Aplicar la estructura:

```bash
ldapadd -x -D "cn=admin,dc=ejemplo,dc=com" -W -f base_estructura.ldif
```

### Paso 1.4: Crear usuarios

Crear archivo `usuarios.ldif`:

```ldif
dn: uid=bzappellini,ou=Docentes,dc=ejemplo,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: bzappellini
cn: Bruno Zappellini
sn: Zappellini
givenName: Bruno
mail: bzappellini@ejemplo.com
uidNumber: 10001
gidNumber: 10001
homeDirectory: /home/bzappellini
loginShell: /bin/bash
userPassword: {SSHA}[hash_generado]

dn: uid=alumno1,ou=Alumnos,dc=ejemplo,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: alumno1
cn: Alumno Uno
sn: Uno
givenName: Alumno
mail: alumno1@ejemplo.com
uidNumber: 20001
gidNumber: 20001
homeDirectory: /home/alumno1
loginShell: /bin/bash
userPassword: {SSHA}[hash_generado]
```

Generar hash de contraseña:

```bash
slappasswd
# Ingresar la contraseña cuando se solicite
# Copiar el hash generado y reemplazar [hash_generado] en el archivo
```

Agregar usuarios:

```bash
ldapadd -x -D "cn=admin,dc=ejemplo,dc=com" -W -f usuarios.ldif
```

### Paso 1.5: Búsqueda y autenticación

```bash
# Buscar todos los usuarios
ldapsearch -x -b "dc=ejemplo,dc=com"

# Buscar usuarios en la OU de Docentes
ldapsearch -x -b "ou=Docentes,dc=ejemplo,dc=com"

# Buscar un usuario específico
ldapsearch -x -b "dc=ejemplo,dc=com" "(cn=Bruno Zappellini)"

# Autenticar como usuario
ldapwhoami -x -D "uid=bzappellini,ou=Docentes,dc=ejemplo,dc=com" -W
```

### Análisis de resultados

**Puntos a analizar:**
- Estructura del DN (Distinguished Name)
- Atributos de cada entrada (cn, sn, uid, mail)
- Jerarquía organizacional (dc → ou → uid)
- Verificación de usuarios autenticados

---

## Parte 2: Kerberos (30 min)

### Objetivo

Comprender la autenticación basada en tickets y la gestión de credenciales.

### Paso 2.1: Instalación del KDC

```bash
# Instalar servidor Kerberos
sudo apt install krb5-kdc krb5-admin-server -y
```

Durante la instalación:
- Default Kerberos realm: `EJEMPLO.COM`
- Servidor Kerberos: `kdc.ejemplo.com`
- Servidor administrativo: `kdc.ejemplo.com`

### Paso 2.2: Configurar el realm

```bash
# Crear la base de datos del realm
sudo krb5_newrealm
# Ingresar contraseña maestra cuando se solicite
```

### Paso 2.3: Crear principales

```bash
# Ingresar al modo administrativo
sudo kadmin.local

# Dentro de kadmin.local:
addprinc bzappellini
addprinc alumno1
addprinc -randkey host/servidor.ejemplo.com

# Listar principales
listprincs

# Salir
quit
```

### Paso 2.4: Configurar cliente Kerberos

Instalar cliente:

```bash
sudo apt install krb5-user -y
```

Configurar `/etc/krb5.conf`:

```ini
[libdefaults]
    default_realm = EJEMPLO.COM
    dns_lookup_realm = false
    dns_lookup_kdc = false

[realms]
    EJEMPLO.COM = {
        kdc = kdc.ejemplo.com
        admin_server = kdc.ejemplo.com
    }

[domain_realm]
    .ejemplo.com = EJEMPLO.COM
    ejemplo.com = EJEMPLO.COM
```

### Paso 2.5: Autenticación con Kerberos

```bash
# Obtener ticket para bzappellini
kinit bzappellini
# Ingresar contraseña cuando se solicite

# Listar tickets actuales
klist

# Ver información detallada del ticket
klist -e

# Renovar ticket
kinit -R

# Destruir tickets
kdestroy
```

### Análisis de tickets

**Comandos de análisis:**

```bash
# Obtener ticket y ver detalles
kinit bzappellini
klist -f

# Observar:
# - Valid starting: fecha/hora de inicio
# - Expires: fecha/hora de expiración
# - Renew until: límite para renovación
# - Ticket flags: propiedades del ticket
```

**Análisis sugerido:**
- Observar el contenido del ticket
- Analizar los tiempos de expiración
- Probar autenticaciones repetidas
- Verificar renovación del ticket
- Comprobar sincronización horaria

---

## Ejercicios Adicionales

### Ejercicio 1: Modificar usuarios en LDAP

Crear archivo `modificar_usuario.ldif`:

```ldif
dn: uid=bzappellini,ou=Docentes,dc=ejemplo,dc=com
changetype: modify
replace: mail
mail: bruno.zappellini@unpsjb.edu.ar
```

Aplicar cambios:

```bash
ldapmodify -x -D "cn=admin,dc=ejemplo,dc=com" -W -f modificar_usuario.ldif
```

### Ejercicio 2: Grupos en LDAP

Crear archivo `grupos.ldif`:

```ldif
dn: cn=profesores,ou=Grupos,dc=ejemplo,dc=com
objectClass: groupOfNames
cn: profesores
member: uid=bzappellini,ou=Docentes,dc=ejemplo,dc=com

dn: cn=estudiantes,ou=Grupos,dc=ejemplo,dc=com
objectClass: groupOfNames
cn: estudiantes
member: uid=alumno1,ou=Alumnos,dc=ejemplo,dc=com
```

```bash
ldapadd -x -D "cn=admin,dc=ejemplo,dc=com" -W -f grupos.ldif
```

### Ejercicio 3: Políticas de tickets en Kerberos

```bash
sudo kadmin.local

# Modificar política de expiración
modprinc -maxlife "8 hours" bzappellini

# Modificar periodo de renovación
modprinc -maxrenewlife "7 days" bzappellini

# Verificar políticas
getprinc bzappellini

quit
```

---

## Verificación y Evaluación

### Checklist de Verificación

- [ ] OpenLDAP instalado y corriendo
- [ ] Estructura de OUs creada correctamente
- [ ] Al menos 2 usuarios creados en LDAP
- [ ] Búsquedas LDAP funcionan correctamente
- [ ] Autenticación LDAP exitosa
- [ ] KDC instalado y configurado
- [ ] Principales creados en Kerberos
- [ ] Tickets obtenidos correctamente con `kinit`
- [ ] `klist` muestra información de tickets
- [ ] Tickets pueden ser destruidos con `kdestroy`

### Preguntas de Evaluación

1. **¿Qué es un Distinguished Name (DN) en LDAP y cómo se estructura?**

2. **¿Cuál es la diferencia entre autenticación con contraseñas tradicionales y autenticación con tickets de Kerberos?**

3. **¿Por qué es importante la sincronización horaria en Kerberos?**

4. **¿Qué ventajas ofrece LDAP como servicio de directorio centralizado?**

5. **Explica el flujo básico de obtención de un TGT en Kerberos.**

---

## Solución de Problemas

### Error: "Can't contact LDAP server"

```bash
# Verificar que slapd está corriendo
sudo systemctl status slapd

# Reiniciar servicio si es necesario
sudo systemctl restart slapd

# Verificar logs
sudo journalctl -u slapd -n 50
```

### Error: "Clock skew too great" en Kerberos

```bash
# Instalar NTP
sudo apt install ntp -y

# Sincronizar tiempo
sudo ntpdate -s time.nist.gov

# Verificar hora actual
date
```

### Error: "Invalid credentials" en LDAP

- Verificar que la contraseña esté correcta
- Comprobar que el DN del usuario sea exacto
- Revisar que el hash de contraseña se generó correctamente

---

## Conclusiones

Al completar este laboratorio, has:

✅ Instalado y configurado un servicio de directorio LDAP  
✅ Creado y gestionado usuarios y grupos en LDAP  
✅ Comprendido la estructura jerárquica de directorios  
✅ Configurado un Key Distribution Center de Kerberos  
✅ Trabajado con tickets de autenticación  
✅ Entendido las diferencias entre autenticación tradicional y basada en tickets

**Próximos pasos:**
- Laboratorio 3: OAuth 2.0 y OpenID Connect con Keycloak
- Integrar LDAP como backend de autenticación para aplicaciones
- Explorar integración LDAP + Kerberos en Active Directory
