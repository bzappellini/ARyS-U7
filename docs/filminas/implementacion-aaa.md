# Implementación de AAA
## Casos Prácticos y Configuración

---

## Agenda

1. Instalación de FreeRADIUS
2. Configuración básica
3. Integración con sistemas
4. Casos prácticos
5. Troubleshooting

---

## Instalación de FreeRADIUS

### En Debian/Ubuntu:
```bash
sudo apt update
sudo apt install freeradius freeradius-utils
```

### En CentOS/RHEL:
```bash
sudo yum install freeradius freeradius-utils
```

---

## Estructura de directorios

```
/etc/freeradius/
├── clients.conf      # Clientes RADIUS (NAS)
├── users            # Base de datos local usuarios
├── radiusd.conf     # Configuración principal
├── proxy.conf       # Configuración de proxy
├── sites-enabled/   # Sitios virtuales activos
│   └── default
└── mods-enabled/    # Módulos activos
    ├── eap
    ├── sql
    └── ldap
```

---

## Configuración de Clientes

**Archivo**: `/etc/freeradius/clients.conf`

```conf
client switch-oficina {
    ipaddr = 192.168.1.10
    secret = MiSecretoSuperSeguro123!
    shortname = switch-01
    nastype = cisco
}

client ap-wireless {
    ipaddr = 192.168.1.20
    secret = OtroSecretoSeguro456!
    shortname = ap-01
}
```

---

## Configuración de Usuarios

**Archivo**: `/etc/freeradius/users`

```conf
# Usuario con acceso completo
admin Cleartext-Password := "admin123"
    Service-Type = Administrative-User

# Usuario con acceso limitado
usuario1 Cleartext-Password := "pass123"
    Service-Type = Framed-User,
    Framed-IP-Address = 192.168.100.10

# Usuario con tiempo de sesión limitado
temporal Cleartext-Password := "temp123"
    Session-Timeout = 3600
```

---

## Integración con LDAP

**Archivo**: `/etc/freeradius/mods-enabled/ldap`

```conf
ldap {
    server = "ldap.empresa.com"
    port = 389
    identity = "cn=admin,dc=empresa,dc=com"
    password = "password"
    base_dn = "ou=users,dc=empresa,dc=com"
    
    user {
        filter = "(uid=%{%{Stripped-User-Name}:-%{User-Name}})"
        base_dn = "${..base_dn}"
    }
}
```

---

## Integración con Active Directory

```conf
ldap {
    server = "ad.empresa.com"
    port = 389
    identity = "cn=radiusadmin,cn=users,dc=empresa,dc=com"
    password = "password"
    base_dn = "dc=empresa,dc=com"
    
    user {
        filter = "(sAMAccountName=%{%{User-Name}})"
    }
}
```

---

## Integración con SQL (MySQL)

**Archivo**: `/etc/freeradius/mods-enabled/sql`

```conf
sql {
    driver = "rlm_sql_mysql"
    dialect = "mysql"
    
    server = "localhost"
    port = 3306
    login = "radius"
    password = "radiuspass"
    
    radius_db = "radius"
}
```

---

## Base de datos RADIUS

```sql
CREATE TABLE radcheck (
    id int(11) NOT NULL auto_increment,
    username varchar(64) NOT NULL,
    attribute varchar(64) NOT NULL,
    op char(2) NOT NULL DEFAULT '==',
    value varchar(253) NOT NULL,
    PRIMARY KEY (id)
);

INSERT INTO radcheck (username, attribute, op, value)
VALUES ('testuser', 'Cleartext-Password', ':=', 'testpass');
```

---

## Caso Práctico 1: WiFi 802.1X

### Componentes:
- Access Point con soporte 802.1X
- FreeRADIUS como servidor de autenticación
- Certificados para EAP-TLS

### Configuración:
```conf
# En el AP (WPA2-Enterprise)
SSID: Empresa-Segura
Security: WPA2-Enterprise
RADIUS Server: 192.168.1.100
RADIUS Port: 1812
RADIUS Secret: MiSecretoSeguro
```

---

## Caso Práctico 2: VPN con RADIUS

### Integración con OpenVPN:

```conf
# En OpenVPN server.conf
plugin /usr/lib/openvpn/radiusplugin.so \
    /etc/openvpn/radiusplugin.cnf

# En radiusplugin.cnf
NAS-Identifier=OpenVPN
Service-Type=5
NAS-Port-Type=5
NAS-IP-Address=192.168.1.1
server {
    acctport=1813
    authport=1812
    name=192.168.1.100
    sharedsecret=VPNSecret123
}
```

---

## Caso Práctico 3: Administración de Switches

### Configuración en Cisco IOS:

```cisco
! Configurar servidor RADIUS
aaa new-model
radius-server host 192.168.1.100 auth-port 1812 
radius-server key MiSecretoSeguro

! Configurar AAA para login
aaa authentication login default group radius local
aaa authorization exec default group radius local
aaa accounting exec default start-stop group radius
```

---

## EAP (Extensible Authentication Protocol)

### Métodos soportados:

- **EAP-TLS**: Certificados mutuos (más seguro)
- **EAP-TTLS**: Túnel TLS con autenticación interna
- **EAP-PEAP**: Protected EAP (similar a TTLS)
- **EAP-MD5**: Simple pero inseguro (evitar)

---

## Configuración EAP-TLS

**Archivo**: `/etc/freeradius/mods-enabled/eap`

```conf
eap {
    default_eap_type = tls
    
    tls-config tls-common {
        private_key_file = /etc/freeradius/certs/server.key
        certificate_file = /etc/freeradius/certs/server.pem
        ca_file = /etc/freeradius/certs/ca.pem
        dh_file = /etc/freeradius/certs/dh
        random_file = /dev/urandom
    }
}
```

---

## Testing y Debugging

### Modo debug:
```bash
sudo freeradius -X
```

### Probar autenticación:
```bash
radtest usuario password localhost 0 testing123

# Con EAP:
eapol_test -c test.conf -a 192.168.1.100 -s secret
```

---

## Logs y Monitoreo

### Archivos de log:
```
/var/log/freeradius/radius.log
/var/log/freeradius/radacct/
```

### Comandos útiles:
```bash
# Ver logs en tiempo real
tail -f /var/log/freeradius/radius.log

# Verificar servicio
systemctl status freeradius

# Verificar sintaxis de configuración
freeradius -C
```

---

## Troubleshooting común

### Problema: Cliente no puede conectar
- ✓ Verificar secret compartido
- ✓ Verificar firewall (UDP 1812, 1813)
- ✓ Verificar IP del cliente en clients.conf

### Problema: Autenticación falla
- ✓ Verificar credenciales de usuario
- ✓ Revisar logs en modo debug
- ✓ Verificar configuración de módulos (LDAP, SQL)

---

## Alta Disponibilidad

### Configuración de redundancia:

1. **Múltiples servidores RADIUS**
2. **Balanceo de carga**
3. **Sincronización de configuración**
4. **Backup de bases de datos**

```conf
# En el cliente (AP, Switch)
Primary RADIUS: 192.168.1.100
Secondary RADIUS: 192.168.1.101
```

---

## Mejores Prácticas

1. 🔐 Usar secretos fuertes y únicos
2. 📝 Documentar todas las configuraciones
3. 🔄 Implementar redundancia
4. 📊 Monitorear logs regularmente
5. 🔒 Usar EAP-TLS cuando sea posible
6. 🛡️ Mantener sistema actualizado
7. 📈 Auditar accesos periódicamente

---

## Herramientas adicionales

- **daloRADIUS**: Interface web para FreeRADIUS
- **phpRADmin**: Gestión de usuarios vía web
- **RADIUSdesk**: Plataforma completa de gestión
- **PacketFence**: NAC con RADIUS integrado

---

## Recursos y Referencias

- 📚 [FreeRADIUS Documentation](https://freeradius.org/documentation/)
- 📖 [RFC 2865 - RADIUS](https://tools.ietf.org/html/rfc2865)
- 🌐 [Wiki de FreeRADIUS](https://wiki.freeradius.org/)
- 💬 [Lista de correo FreeRADIUS](http://lists.freeradius.org/)

---

## Laboratorio Práctico

### Ejercicios propuestos:

1. Instalar y configurar FreeRADIUS
2. Crear usuarios locales y probar autenticación
3. Integrar con LDAP/AD
4. Configurar un AP para 802.1X
5. Implementar contabilidad (accounting)
6. Analizar logs y resolver problemas

---

# ¿Preguntas?

📧 Contacto
🔗 [Volver al índice](../docs/index.html)
