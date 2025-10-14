# Protocolos AAA
## RADIUS, TACACS+ y Diameter

---

## RADIUS

### Remote Authentication Dial-In User Service

- Protocolo estándar de facto para AAA
- RFC 2865 (Authentication) y RFC 2866 (Accounting)
- Puerto UDP 1812 (auth) y 1813 (accounting)

---

## Características de RADIUS

- ✅ Protocolo abierto y estándar
- ✅ Amplio soporte de fabricantes
- ✅ Cifrado de contraseñas
- ❌ Solo la contraseña se cifra (no todo el paquete)
- ❌ Combina autenticación y autorización

---

## TACACS+

### Terminal Access Controller Access Control System Plus

- Protocolo propietario de Cisco
- Mejora sobre TACACS y XTACACS
- Puerto TCP 49

---

## Características de TACACS+

- ✅ Cifrado completo del payload
- ✅ Separa AAA (autenticación, autorización, accounting)
- ✅ Mayor control granular
- ✅ Soporte multiprotocolo
- ❌ Principalmente Cisco (limitado en otros fabricantes)

---

## RADIUS vs TACACS+

| Característica | RADIUS | TACACS+ |
|----------------|--------|---------|
| Protocolo transporte | UDP | TCP |
| Cifrado | Solo contraseña | Payload completo |
| Separación AAA | No | Sí |
| Estándar | Abierto | Propietario Cisco |
| Uso principal | ISP, WiFi | Administración dispositivos |

---

## Diameter

### Evolución de RADIUS

- Sucesor de RADIUS
- RFC 6733
- Puerto TCP/SCTP 3868

---

## Características de Diameter

- ✅ Protocolo basado en TCP o SCTP
- ✅ Seguridad mejorada (IPSec, TLS)
- ✅ Mejor detección de errores
- ✅ Soporte para roaming
- ✅ Diseñado para redes móviles (4G, 5G)

---

## Comparación de Protocolos

```
Complejidad y Capacidades

TACACS+  ████████░░  Cisco, Admin red
RADIUS   ██████░░░░  ISP, WiFi, VPN
Diameter ██████████  Móviles, IMS, 5G
```

---

## Implementaciones Open Source

### RADIUS
- **FreeRADIUS**: Más popular y completo
- **OpenRADIUS**: Ligero
- Radiator

### TACACS+
- **tac_plus**: Implementación de Shrubbery Networks
- **FreeTACACS**: Derivado de tac_plus

---

## Flujo de Autenticación RADIUS

```
1. Usuario → NAS: Solicitud de acceso
2. NAS → RADIUS: Access-Request
3. RADIUS: Valida credenciales
4. RADIUS → NAS: Access-Accept/Reject
5. NAS → Usuario: Concede/Niega acceso
6. NAS → RADIUS: Accounting-Request (Start)
7. ... sesión activa ...
8. NAS → RADIUS: Accounting-Request (Stop)
```

---

## Atributos RADIUS

### Ejemplos comunes:
- User-Name
- User-Password
- NAS-IP-Address
- Service-Type
- Framed-IP-Address
- Session-Timeout
- Acct-Session-Id

---

## Seguridad en AAA

### Mejores prácticas:

1. Usar secretos compartidos fuertes
2. Cifrar comunicaciones (IPSec/TLS)
3. Implementar redundancia
4. Monitorear y auditar
5. Actualizar regularmente

---

## Casos de uso por protocolo

**RADIUS**:
- WiFi empresarial (802.1X)
- VPN
- Acceso ISP

**TACACS+**:
- Administración de routers/switches
- Control de comandos en dispositivos

**Diameter**:
- Redes móviles 4G/5G
- IMS (IP Multimedia Subsystem)
- Roaming internacional

---

# Práctica

En la siguiente sesión implementaremos:
- Servidor FreeRADIUS
- Configuración de clientes
- Integración con Active Directory
- Pruebas de autenticación

---

# ¿Preguntas?

📧 Contacto
🔗 [Volver al índice](../docs/index.html)
