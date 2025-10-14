# Introducción a los Servicios AAA

## Conceptos Fundamentales

**AAA** es un acrónimo que representa tres funciones esenciales en seguridad de redes:

- **Authentication (Autenticación)**: Verificación de identidad
- **Authorization (Autorización)**: Control de permisos y recursos
- **Accounting (Auditoría)**: Registro y seguimiento de actividades

## Historia y Evolución

Los servicios AAA surgieron en la década de 1990 con la proliferación de los proveedores de servicios de Internet (ISP) que necesitaban:

1. Autenticar usuarios dial-up
2. Controlar el acceso a recursos
3. Facturar por tiempo de conexión

### Evolución Temporal

- **1990s**: RADIUS y TACACS para ISPs
- **2000s**: Adopción en redes empresariales y WiFi
- **2010s**: Integración con cloud y móviles
- **2020s**: Zero Trust y autenticación continua

## Importancia en la Seguridad Moderna

### Desafíos Actuales

1. **Perímetro Difuso**: Trabajo remoto, cloud, BYOD
2. **Amenazas Sofisticadas**: Ataques dirigidos, ransomware
3. **Cumplimiento Normativo**: GDPR, HIPAA, PCI-DSS
4. **Escalabilidad**: Miles o millones de usuarios

### Beneficios de AAA

- ✅ **Centralización**: Un punto de control para toda la organización
- ✅ **Consistencia**: Políticas uniformes en todos los sistemas
- ✅ **Visibilidad**: Auditoría completa de accesos y actividades
- ✅ **Agilidad**: Provisioning y deprovisioning rápido
- ✅ **Cumplimiento**: Facilita auditorías y reportes regulatorios

## Arquitectura Básica

```
┌──────────────┐
│   Usuario    │
│   Dispositivo│
└──────┬───────┘
       │
       ▼
┌──────────────┐
│     NAS      │ ← Network Access Server
│ (AP, Switch, │   (Punto de aplicación de políticas)
│  VPN, etc.)  │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Servidor AAA │
│  (RADIUS,    │
│   TACACS+,   │
│  Diameter)   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│   Backend    │
│(LDAP, AD,    │
│ SQL, etc.)   │
└──────────────┘
```

## Componentes Principales

### 1. Supplicant (Cliente)
- Usuario o dispositivo que solicita acceso
- Proporciona credenciales
- Ejemplos: PC, smartphone, IoT device

### 2. Authenticator (NAS)
- Intermediario entre cliente y servidor AAA
- Aplica decisiones de acceso
- Ejemplos: Switch, AP WiFi, VPN gateway

### 3. Authentication Server (Servidor AAA)
- Procesa solicitudes de autenticación
- Consulta backend de identidades
- Toma decisiones de autorización
- Registra eventos de auditoría

### 4. Identity Store (Backend)
- Almacena credenciales y atributos
- Ejemplos: Active Directory, LDAP, SQL

## Flujo Básico de Operación

### 1. Autenticación
```
1. Usuario solicita acceso
2. NAS reenvía credenciales a servidor AAA
3. Servidor valida contra backend
4. Servidor responde: Accept/Reject
5. NAS concede o niega acceso
```

### 2. Autorización
```
1. Usuario autenticado solicita recurso
2. Servidor AAA consulta políticas
3. Determina permisos aplicables
4. Comunica decisión al NAS
5. NAS aplica restricciones
```

### 3. Auditoría
```
1. NAS reporta inicio de sesión
2. Actividad continua se registra
3. NAS reporta fin de sesión
4. Servidor almacena en logs/BD
5. Disponible para análisis
```

## Casos de Uso Comunes

### Acceso a Red
- WiFi corporativo (802.1X)
- Switches con autenticación por puerto
- VPN de acceso remoto

### Administración de Dispositivos
- Login a routers y switches
- Acceso a firewalls
- Gestión de servidores

### Aplicaciones
- Single Sign-On (SSO)
- Aplicaciones web
- APIs y microservicios

### Servicios
- ISP y acceso a Internet
- Hotspots públicos
- Redes móviles (4G/5G)

## Consideraciones de Diseño

### Seguridad
- Cifrado de comunicaciones
- Secretos compartidos robustos
- Segregación de tráfico AAA
- Protección del servidor

### Disponibilidad
- Redundancia de servidores
- Failover automático
- Replicación de datos
- Monitoreo continuo

### Escalabilidad
- Dimensionamiento adecuado
- Balanceo de carga
- Distribución geográfica
- Caché de autenticaciones

### Integración
- Compatibilidad con sistemas existentes
- APIs y estándares abiertos
- Federación de identidades
- Sincronización con HR

## Próximos Pasos

En los siguientes capítulos exploraremos en detalle:

1. Autenticación: Métodos y mejores prácticas
2. Autorización: Modelos y políticas
3. Auditoría: Logging y análisis
4. Protocolos: RADIUS, TACACS+, Diameter
5. Implementación: Casos prácticos
6. Seguridad: Hardening y protección

## Recursos Adicionales

- [RFC 2865 - RADIUS](https://tools.ietf.org/html/rfc2865)
- [RFC 6733 - Diameter](https://tools.ietf.org/html/rfc6733)
- [NIST SP 800-63 - Digital Identity Guidelines](https://pages.nist.gov/800-63-3/)
- [FreeRADIUS Documentation](https://freeradius.org/documentation/)
