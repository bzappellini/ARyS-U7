# Servicios de Autenticación y Directorios

## LDAP (Lightweight Directory Access Protocol)

LDAP es un protocolo que organiza la información jerárquicamente, similar a un árbol. Es fundamental para la gestión centralizada de identidades en organizaciones.

### Características Principales

- **Estructura jerárquica**: Organización en forma de árbol (DIT - Directory Information Tree)
- **Entradas DN**: Distinguished Names únicos para cada objeto
- **Atributos**: mail, uid, cn, ou, dc, etc.
- **Operaciones**: bind, search, modify, add, delete

### Ejemplo de DN

```
cn=Bruno Zappellini,ou=Docentes,dc=unpsjb,dc=edu,dc=ar
```

Donde:
- `cn` = Common Name
- `ou` = Organizational Unit
- `dc` = Domain Component

### Comandos Básicos

**Búsqueda de usuarios:**
```bash
ldapsearch -x -b "dc=unpsjb,dc=edu,dc=ar" "(cn=Bruno Zappellini)"
```

**Autenticación:**
```bash
ldapwhoami -x -D "cn=usuario,dc=ejemplo,dc=com" -W
```

### Implementaciones Comunes

- **OpenLDAP**: Implementación open source más popular
- **Active Directory**: Implementación de Microsoft que combina LDAP + Kerberos
- **FreeIPA**: Solución integrada de Red Hat
- **389 Directory Server**: Proyecto de la comunidad Fedora

### Concepto Clave

⚠️ LDAP no autentica por sí solo — sólo almacena usuarios y credenciales. Otros servicios (Kerberos, RADIUS, aplicaciones) consultan el directorio para validar identidades.

---

## RADIUS (Remote Authentication Dial-In User Service)

RADIUS es un protocolo cliente-servidor que implementa el modelo AAA completo para control de acceso a redes.

### Características

- **Protocolo**: UDP (puertos 1812 para auth, 1813 para accounting)
- **Arquitectura**: Cliente (NAS) → Servidor RADIUS → Backend (LDAP/DB)
- **Cifrado**: Solo la contraseña se cifra (con shared secret)
- **Estado**: Protocolo sin estado (stateless)

### Casos de Uso

1. **WiFi empresarial (802.1X)**
   - Access Point actúa como NAS
   - Usuario se autentica vía EAP
   - RADIUS valida contra LDAP/AD

2. **VPN**
   - Gateway VPN es el cliente RADIUS
   - Usuarios remotos autentican centralizadamente

3. **Switches y Routers**
   - Autenticación de administradores
   - Autorización de comandos por perfil

### Flujo de Autenticación

```
1. Cliente/Supplicant → NAS: Credenciales
2. NAS → RADIUS Server: Access-Request
3. RADIUS → Backend: Validación
4. RADIUS → NAS: Access-Accept/Reject
5. NAS → Cliente: Acceso permitido/denegado
```

### Implementaciones

- **FreeRADIUS**: Más popular, open source
- **Microsoft NPS**: Network Policy Server (Windows)
- **Cisco ISE**: Identity Services Engine (comercial)

### RADIUS vs TACACS+

| Característica | RADIUS | TACACS+ |
|----------------|--------|---------|
| Protocolo | UDP | TCP |
| Cifrado | Solo password | Todo el payload |
| AAA | Combinado | Separado |
| Origen | Estándar abierto | Cisco (propietario) |
| Uso principal | Acceso de red | Admin de dispositivos |

---

## Active Directory (AD)

Active Directory es el sistema de gestión de identidades y directorios de Microsoft, ampliamente usado en entornos empresariales.

### Arquitectura

Active Directory combina:
- **LDAP**: Para almacenamiento y consulta de objetos
- **Kerberos**: Para autenticación segura
- **DNS**: Para localización de servicios
- **GPO**: Group Policy Objects para políticas

### Componentes Principales

**1. Dominio**
- Límite de seguridad y replicación
- Controladores de dominio (DC) que replican datos
- Cada DC tiene un KDC (Key Distribution Center)

**2. Bosque (Forest)**
- Colección de dominios que comparten esquema
- Relaciones de confianza entre dominios
- Primer dominio = Forest Root

**3. Objetos**
- Usuarios (User objects)
- Computadoras (Computer objects)
- Grupos (Group objects)
- Unidades Organizacionales (OUs)

### Funcionalidades Clave

**Autenticación**
- Login en dominio usando Kerberos
- Single Sign-On dentro del dominio
- Autenticación mutua cliente-servidor

**Autorización**
- Control de acceso a recursos
- Permisos heredables en OUs
- Delegación de administración

**Políticas (GPO)**
- Configuraciones de seguridad
- Instalación de software
- Scripts de inicio/apagado
- Restricciones de usuario

### Servicios Integrados

- **DNS**: Localización de DCs y servicios
- **DHCP**: Asignación dinámica de IPs
- **Certificate Services**: PKI corporativa
- **Rights Management**: Protección de documentos

### Concepto Clave

💡 AD no sólo autentica — también autoriza y aplica políticas a todos los dispositivos del dominio. Es la columna vertebral de la seguridad en redes Windows.

---

## Kerberos

Kerberos es un protocolo de autenticación de red que usa criptografía de clave simétrica y tickets para autenticar de forma segura.

### Principios de Diseño

- **Sin transmisión de contraseñas**: Se usan tickets cifrados
- **Autenticación mutua**: Cliente y servidor se verifican mutuamente
- **Single Sign-On**: Un login, múltiples servicios
- **Timestamping**: Protección contra replay attacks

### Componentes

**KDC (Key Distribution Center)**
- **AS (Authentication Service)**: Valida usuario, emite TGT
- **TGS (Ticket Granting Service)**: Emite tickets de servicio

**Cliente**
- Usuario o aplicación que solicita autenticación

**Servicio**
- Recurso al que se desea acceder

### Flujo de Autenticación

```
1. Cliente → AS: "Soy usuario X, quiero autenticarme"
2. AS → Cliente: TGT cifrado con clave del usuario
3. Cliente descifra TGT con su contraseña
4. Cliente → TGS: "Quiero acceder al servicio Y" + TGT
5. TGS → Cliente: Service Ticket para Y
6. Cliente → Servicio Y: Service Ticket
7. Servicio Y valida ticket y permite acceso
```

### Tickets

**TGT (Ticket-Granting Ticket)**
- Válido por 8-10 horas típicamente
- Se puede renovar sin reingresar contraseña
- Almacenado en caché de credenciales

**Service Ticket**
- Específico para un servicio
- Tiempo de vida más corto
- Contiene información de autorización

### Comandos Linux/Unix

```bash
# Obtener TGT
kinit usuario@DOMINIO.COM

# Listar tickets
klist

# Ver detalles
klist -e

# Renovar ticket
kinit -R

# Destruir tickets
kdestroy
```

### Comandos Windows

```cmd
# Listar tickets
klist

# Purgar tickets
klist purge

# Obtener tickets (automático al login)
```

### Requisitos y Limitaciones

**Requisitos:**
- ⏰ **Sincronización horaria precisa** (tolerancia típica: 5 minutos)
- 🔒 **Protección del KDC** (punto central de confianza)
- 🔐 **Contraseñas seguras** (base de toda la seguridad)

**Limitaciones:**
- Vulnerabilidades de contraseñas débiles
- Ataques de pass-the-ticket si se compromete ticket
- Sincronización horaria crítica
- KDC es punto único de fallo (requiere HA)

### Kerberos en Active Directory

En AD, Kerberos es el protocolo de autenticación predeterminado:

- Login en dominio = obtención de TGT
- Acceso a recursos = service tickets automáticos
- Delegación Kerberos para aplicaciones web
- Service Principal Names (SPNs) para servicios

---

## Comparación de Tecnologías

| Aspecto | LDAP | RADIUS | Kerberos | Active Directory |
|---------|------|--------|----------|------------------|
| **Función principal** | Directorio | AAA para red | Autenticación | Todo integrado |
| **Protocolo** | TCP 389/636 | UDP 1812/1813 | TCP/UDP 88 | Múltiple |
| **Autenticación** | Bind | EAP/PAP/CHAP | Tickets | Kerberos+NTLM |
| **Caso de uso** | Base de usuarios | WiFi/VPN | SSO corporativo | Dominio Windows |
| **Escalabilidad** | Alta | Alta | Media | Alta |
| **Complejidad** | Media | Baja | Alta | Alta |

---

## Mejores Prácticas

### LDAP
- ✅ Usar LDAPS (LDAP sobre SSL/TLS) para cifrado
- ✅ Implementar políticas de contraseñas robustas
- ✅ Limitar acceso de bind con permisos mínimos
- ✅ Replicación para alta disponibilidad

### RADIUS
- ✅ Shared secrets fuertes y únicos por NAS
- ✅ Usar EAP-TLS con certificados cuando sea posible
- ✅ Habilitar accounting para auditoría
- ✅ Servidores proxy RADIUS para segmentación

### Kerberos
- ✅ Sincronización horaria con NTP
- ✅ Alta disponibilidad de KDCs
- ✅ Rotación de claves del KDC
- ✅ Monitoreo de tickets anómalos

### Active Directory
- ✅ Múltiples DCs por sitio
- ✅ Backups regulares de System State
- ✅ Auditoría de cambios en AD
- ✅ Implementar LAPS para contraseñas locales
- ✅ Deshabilitar NTLM cuando sea posible

---

## Recursos Adicionales

### RFCs y Estándares
- RFC 4511 - LDAP v3
- RFC 2865 - RADIUS Authentication
- RFC 2866 - RADIUS Accounting
- RFC 4120 - Kerberos v5

### Documentación
- [OpenLDAP Documentation](https://www.openldap.org/doc/)
- [FreeRADIUS Documentation](https://freeradius.org/documentation/)
- [MIT Kerberos Documentation](https://web.mit.edu/kerberos/)
- [Microsoft AD Documentation](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/)

### Herramientas
- **ldapsearch, ldapmodify** - Utilidades LDAP
- **radtest** - Cliente RADIUS de prueba
- **kinit, klist, kdestroy** - Herramientas Kerberos
- **Wireshark** - Análisis de protocolos
- **Apache Directory Studio** - GUI para LDAP
