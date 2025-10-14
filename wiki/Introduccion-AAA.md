# Introducción a los Servicios AAA

## Definición

**AAA** es un framework de seguridad que agrupa tres funciones esenciales:

- **Authentication (Autenticación)**: ¿Quién eres?
- **Authorization (Autorización)**: ¿Qué puedes hacer?
- **Accounting (Auditoría)**: ¿Qué hiciste?

## Contexto Histórico

Los servicios AAA surgieron en los años 90 con la necesidad de los ISP (Internet Service Providers) de controlar el acceso dial-up y facturar por tiempo de uso.

### Evolución

| Década | Hito |
|--------|------|
| 1990s | Creación de RADIUS para ISPs |
| 1993 | TACACS+ de Cisco para administración de equipos |
| 2000s | Adopción masiva en WiFi empresarial (802.1X) |
| 2010s | Diameter para redes móviles 4G/LTE |
| 2020s | Integración con Zero Trust y autenticación continua |

## Componentes del Framework AAA

### 1. Autenticación (Authentication)

Proceso de verificar que una entidad es quien dice ser.

**Métodos comunes:**
- Contraseñas
- Certificados digitales
- Tokens de hardware/software
- Biometría
- Autenticación multifactor (MFA)

### 2. Autorización (Authorization)

Determina qué puede hacer un usuario o dispositivo autenticado.

**Elementos de autorización:**
- Acceso a recursos específicos
- Nivel de privilegios
- Comandos permitidos
- Restricciones temporales
- Asignación de VLAN

### 3. Auditoría (Accounting)

Registro detallado de las actividades para:
- Facturación
- Análisis de seguridad
- Cumplimiento normativo
- Troubleshooting
- Análisis de capacidad

## Arquitectura Básica

```
┌─────────────┐
│   Cliente   │ ← Supplicant (Usuario/Dispositivo)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│     NAS     │ ← Authenticator (AP, Switch, VPN)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Servidor    │ ← Authentication Server
│    AAA      │   (RADIUS, TACACS+, Diameter)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Backend    │ ← Identity Store
│(LDAP/AD/SQL)│   (Base de datos de usuarios)
└─────────────┘
```

## Protocolos AAA

### RADIUS
- **Protocolo**: UDP
- **Puerto**: 1812 (auth), 1813 (acct)
- **Uso**: ISP, WiFi, VPN
- **Ventaja**: Estándar abierto, amplio soporte

### TACACS+
- **Protocolo**: TCP
- **Puerto**: 49
- **Uso**: Administración de dispositivos
- **Ventaja**: Separación de AAA, cifrado completo

### Diameter
- **Protocolo**: TCP/SCTP
- **Puerto**: 3868
- **Uso**: Redes móviles (4G/5G)
- **Ventaja**: Mayor escalabilidad, mejor seguridad

Ver: [Comparación detallada de Protocolos](Comparacion-Protocolos)

## Casos de Uso

### 1. Acceso a la Red
- Control de acceso WiFi (WPA2/WPA3-Enterprise)
- Autenticación por puerto en switches (802.1X)
- VPN de acceso remoto

### 2. Administración de Equipos
- Login a routers y switches
- Autorización de comandos
- Auditoría de cambios de configuración

### 3. Servicios de Usuario
- Single Sign-On (SSO)
- Portales cautivos (hotspots)
- Acceso a aplicaciones corporativas

### 4. Compliance y Auditoría
- Registro de accesos para SOX, HIPAA, PCI-DSS
- Seguimiento de actividades privilegiadas
- Reportes de uso y acceso

## Beneficios de Implementar AAA

✅ **Centralización**: Un único punto de gestión de identidades  
✅ **Seguridad**: Control granular de acceso  
✅ **Visibilidad**: Auditoría completa de actividades  
✅ **Escalabilidad**: Soporta desde decenas hasta millones de usuarios  
✅ **Flexibilidad**: Políticas personalizables por usuario, grupo, hora, ubicación  
✅ **Cumplimiento**: Facilita adherencia a normativas

## Consideraciones de Implementación

### Seguridad
- Usar secretos compartidos robustos
- Cifrar comunicaciones (IPSec, TLS)
- Segmentar red AAA
- Proteger servidor AAA (hardening, firewall)

### Disponibilidad
- Implementar redundancia (primario/secundario)
- Considerar balanceo de carga
- Monitoreo continuo
- Backups regulares de configuración

### Rendimiento
- Dimensionar adecuadamente el hardware
- Considerar cache de autenticaciones
- Optimizar consultas a backend
- Planificar para crecimiento

## Próximos Pasos

Para profundizar en cada componente del framework AAA:

- [Autenticación en detalle](Autenticacion)
- [Autorización en detalle](Autorizacion)
- [Auditoría en detalle](Auditoria)

Para implementación práctica:

- [Instalación de FreeRADIUS](Instalacion-FreeRADIUS)
- [Configuración Básica](Configuracion-Basica)
- [Trabajos Prácticos](https://bzappellini.github.io/ARyS-U7/practicos.html)

## Referencias

- [RFC 2865 - RADIUS](https://tools.ietf.org/html/rfc2865)
- [RFC 6733 - Diameter Base Protocol](https://tools.ietf.org/html/rfc6733)
- [NIST SP 800-63 - Digital Identity Guidelines](https://pages.nist.gov/800-63-3/)
- [IEEE 802.1X - Port-Based Network Access Control](https://standards.ieee.org/standard/802_1X-2020.html)

---

[← Volver a la Home](Home) | [Siguiente: Autenticación →](Autenticacion)
