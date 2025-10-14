# 📘 Unidad 7 — Sistemas de Autenticación y Control de Acceso (AAA)

**Carrera:** Licenciatura en Sistemas  
**Materia:** Administración de Redes y Seguridad  
**Sede:** Trelew — UNPSJB  
**Duración total:** 3 h teóricas  
**Nivel:** Avanzado (post redes, criptografía y seguridad)  

---

## 🎯 Objetivos

- Comprender los mecanismos modernos de autenticación y autorización.  
- Analizar cómo se implementan y combinan en redes corporativas y sistemas distribuidos.  
- Conocer las tecnologías actuales de federación de identidades y autenticación multifactor.  
- Relacionar los mecanismos de autenticación con modelos de control de acceso (ACL, RBAC, ABAC).  

---

## 🧩 1. Modelo AAA (Autenticación, Autorización, Auditoría)

El modelo **AAA** resume los tres pilares de la seguridad en sistemas informáticos:

| Función | Descripción | Ejemplo |
|----------|-------------|----------|
| **Autenticación (Authentication)** | Verifica la identidad de un usuario o sistema. | Login con usuario y contraseña. |
| **Autorización (Authorization)** | Define qué puede hacer una entidad autenticada. | Acceso sólo de lectura a una base de datos. |
| **Auditoría (Accounting / Auditing)** | Registra las acciones realizadas. | Logs de acceso, tiempo de sesión, IP de origen. |

🧠 **Concepto clave:**  
AAA no es un protocolo, sino una **arquitectura de seguridad**. Protocolos como RADIUS o TACACS+ implementan este modelo en redes.

💡 **Ejemplo:**  
En una red WiFi corporativa:
- Autenticación: el servidor RADIUS valida al usuario.  
- Autorización: se asigna la VLAN correspondiente.  
- Auditoría: se registra tiempo e IP de conexión.

---

## 🧩 2. Servicios de autenticación y directorios

Los sistemas modernos centralizan la autenticación a través de **servicios de directorio**, que guardan identidades y credenciales de todos los usuarios de una organización.

### 🔹 LDAP (Lightweight Directory Access Protocol)

- Protocolo que organiza la información jerárquicamente (similar a un árbol).  
- Define entradas como **DN (Distinguished Name)** y atributos (mail, uid, cn, etc.).  
- Usado por Active Directory, OpenLDAP, FreeIPA, etc.

**Ejemplo de DN:**
```
cn=Bruno Zappellini,ou=Docentes,dc=unpsjb,dc=edu,dc=ar
```

**Comando de búsqueda:**
```bash
ldapsearch -x -b "dc=unpsjb,dc=edu,dc=ar" "(cn=Bruno Zappellini)"
```

🧠 **Concepto clave:** LDAP no autentica por sí solo — sólo almacena usuarios y contraseñas. Otros servicios (Kerberos, RADIUS, AD) consultan el directorio para validar identidades.

---

### 🔹 RADIUS (Remote Authentication Dial-In User Service)

- Implementa el modelo AAA completo (autenticación, autorización y accounting).  
- Usa UDP (puertos 1812 y 1813).  
- Empleado en redes WiFi empresariales, VPNs y routers.  
- Autenticación centralizada con servidores como **FreeRADIUS** o **Microsoft NPS**.

**Arquitectura:**
```
Cliente (NAS) ---> Servidor RADIUS ---> Base de datos / LDAP / AD
```

💡 **Ejemplo:**  
Cuando te conectás al WiFi institucional, el Access Point envía tus credenciales al servidor RADIUS, que las valida en el directorio LDAP o AD.

---

### 🔹 Active Directory (AD)

- Sistema de administración de identidades de Microsoft.  
- Combina **LDAP + Kerberos** y ofrece autenticación centralizada.  
- Gestiona usuarios, equipos, políticas de seguridad (GPO) y permisos.

**Estructura básica:**
- **Bosques (Forests)** → agrupan **dominios**.
- Cada dominio posee un **KDC (Key Distribution Center)** para Kerberos.
- Usa **DNS** para localizar controladores de dominio.

🧠 **Concepto clave:**  
AD no sólo autentica — también autoriza y aplica políticas a todos los dispositivos del dominio.

---

## 🧩 3. Kerberos — Autenticación basada en tickets

### 🔹 Introducción

Kerberos es un protocolo de autenticación diseñado en el MIT que usa **tickets** en lugar de enviar contraseñas por la red.

**Ventajas:**
- Las credenciales nunca se transmiten directamente.
- Autenticación mutua (cliente y servidor).
- Single Sign-On (SSO) dentro del dominio.

### 🔹 Componentes

- **KDC (Key Distribution Center)**: servidor central de confianza
  - **AS (Authentication Service)**: valida usuarios y emite tickets
  - **TGS (Ticket Granting Service)**: emite tickets de servicio
- **Cliente**: usuario o aplicación que solicita autenticación
- **Servicio**: recurso al que se desea acceder

### 🔹 Flujo simplificado

1. **Cliente → AS**: solicita autenticación con usuario/contraseña
2. **AS → Cliente**: devuelve TGT (Ticket-Granting Ticket) cifrado
3. **Cliente → TGS**: presenta TGT y solicita acceso a un servicio
4. **TGS → Cliente**: emite ticket de servicio
5. **Cliente → Servicio**: presenta ticket de servicio y accede al recurso

🧠 **Concepto clave:**  
El TGT permite al usuario obtener tickets de servicio sin volver a ingresar la contraseña. Los tickets tienen **tiempo de expiración** y requieren **sincronización horaria** precisa.

**Comandos útiles (Linux):**
```bash
kinit usuario@DOMINIO.COM     # Obtener TGT
klist                          # Listar tickets actuales
kdestroy                       # Destruir tickets
```

---

## 🧩 4. Autenticación moderna basada en tokens

### 🔹 JWT (JSON Web Token)

**JWT** es un estándar para representar información de identidad de forma compacta y autoverificable.

**Estructura:**
```
Header.Payload.Signature
```

**Ejemplo decodificado:**
```json
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  "sub": "1234567890",
  "name": "Bruno Zappellini",
  "iat": 1516239022,
  "exp": 1516242622
}
```

🧠 **Concepto clave:**  
JWT **no cifra** la información — sólo la **firma**. Cualquiera puede leer el payload, pero sólo quien tiene la clave puede verificar que no fue alterado.

**Validación de JWT:**
```bash
# Verificar en jwt.io o con una biblioteca
curl -H "Authorization: Bearer $TOKEN" https://api.ejemplo.com/recurso
```

---

### 🔹 OAuth 2.0

OAuth 2.0 es un **framework de autorización** (no autenticación) que permite que aplicaciones accedan a recursos en nombre de un usuario sin conocer su contraseña.

**Roles principales:**
- **Resource Owner**: dueño del recurso (usuario)
- **Client**: aplicación que solicita acceso
- **Authorization Server**: emite tokens
- **Resource Server**: protege recursos y valida tokens

**Flujos comunes:**
1. **Authorization Code Flow**: para aplicaciones web con backend
2. **Authorization Code + PKCE**: para aplicaciones móviles/SPA
3. **Client Credentials**: para comunicación entre servicios

💡 **Ejemplo:**  
Una app móvil solicita acceso a tu Google Drive. En lugar de darte tu contraseña de Google, OAuth genera un token temporal con permisos limitados.

---

### 🔹 OpenID Connect (OIDC)

OIDC extiende OAuth 2.0 para **autenticar usuarios** y obtener información de identidad.

**Tokens principales:**
- **ID Token (JWT)**: contiene información sobre la identidad del usuario
- **Access Token**: usado para acceder a APIs
- **Refresh Token**: permite obtener nuevos access tokens

**Flujo básico:**
1. Usuario se autentica en el IdP (Identity Provider)
2. IdP emite ID Token + Access Token
3. Aplicación valida ID Token y obtiene perfil del usuario
4. Aplicación usa Access Token para llamar APIs

**Providers comunes:**
- Keycloak
- Azure AD / Entra ID
- Google Identity
- Auth0, Okta

---

## 🧩 5. Federación de identidades y Single Sign-On (SSO)

### 🔹 Single Sign-On (SSO)

SSO permite a un usuario **autenticarse una vez** y acceder a múltiples aplicaciones sin volver a ingresar credenciales.

**Beneficios:**
- Mejora la experiencia del usuario
- Reduce la fatiga de contraseñas
- Centraliza el control de acceso

### 🔹 Federación de Identidades

La federación permite que organizaciones **compartan identidades** de forma segura entre dominios diferentes.

**Protocolos principales:**
- **SAML 2.0**: XML, usado en empresas tradicionales
- **OpenID Connect**: JSON/JWT, preferido en aplicaciones modernas
- **WS-Federation**: Microsoft, legacy

💡 **Ejemplo:**  
Una universidad puede federar con Google para que los alumnos usen sus cuentas institucionales para acceder a Gmail, Classroom y otras apps.

---

## 🧩 6. Autenticación multifactor (MFA) y Zero Trust

### 🔹 Autenticación Multifactor (MFA)

MFA requiere **dos o más factores** de autenticación:
- **Algo que sabes**: contraseña, PIN
- **Algo que tienes**: teléfono, token de hardware (YubiKey)
- **Algo que eres**: huella digital, reconocimiento facial

**Métodos comunes:**
- **TOTP (Time-based OTP)**: Google Authenticator, Authy
- **Push notifications**: Duo, Microsoft Authenticator
- **FIDO2/WebAuthn**: llaves de seguridad resistentes a phishing

### 🔹 Zero Trust Architecture

**Principio:** "Nunca confíes, siempre verifica"

**Pilares principales:**
1. Verificar explícitamente toda solicitud
2. Usar acceso con mínimos privilegios
3. Asumir que hay brechas de seguridad

**Implementación:**
- Microsegmentación de redes
- Autenticación continua
- Políticas basadas en contexto (ubicación, dispositivo, comportamiento)

---

## 🧩 7. Control de acceso: ACL, RBAC, ABAC

### 🔹 ACL (Access Control Lists)

**Definición:** Lista de permisos asociada a un recurso específico.

**Ejemplo:**
```
archivo.txt:
  - usuario1: lectura, escritura
  - usuario2: solo lectura
  - grupo_admin: todos los permisos
```

**Limitaciones:**
- Difícil de escalar en sistemas grandes
- Administración descentralizada

### 🔹 RBAC (Role-Based Access Control)

**Definición:** Los permisos se asignan a **roles**, y los usuarios se asignan a roles.

**Ejemplo:**
```
Roles:
  - Admin: puede crear, leer, modificar, eliminar usuarios
  - Editor: puede leer y modificar contenido
  - Lector: solo puede leer contenido

Usuarios:
  - bruno@unpsjb.edu.ar → Admin
  - alumno1@unpsjb.edu.ar → Lector
```

**Ventajas:**
- Simplifica la gestión de permisos
- Facilita auditorías
- Reduce errores en asignación de permisos

### 🔹 ABAC (Attribute-Based Access Control)

**Definición:** Las decisiones de acceso se basan en **atributos** del usuario, recurso y contexto.

**Ejemplo de política:**
```
Permitir acceso SI:
  - usuario.departamento == "IT"
  - recurso.clasificacion == "confidencial"
  - hora >= 08:00 AND hora <= 18:00
  - ubicacion == "red_corporativa"
```

**Motores de políticas:**
- **XACML**: estándar XML
- **OPA (Open Policy Agent)**: políticas en Rego
- **Cedar**: de AWS

**Ventajas:**
- Políticas dinámicas y contextuales
- Granularidad extrema
- Adaptable a cambios sin modificar código

---

## 🧩 Conclusiones

Esta unidad integra los tres pilares de la seguridad de identidad: autenticación, autorización y auditoría.  
Los alumnos deben poder explicar cómo se relacionan Kerberos, LDAP, OAuth y los modelos de control de acceso, y aplicar estos conocimientos tanto en entornos corporativos como en servicios web modernos.

**Próximos pasos:**
- Prácticas de laboratorio con OpenLDAP, Kerberos y Keycloak
- Implementación de RBAC en aplicaciones web
- Configuración de SSO empresarial

---

## 📚 Referencias y lecturas recomendadas

- RFC 4511 — Lightweight Directory Access Protocol (LDAP)  
- RFC 6749 — OAuth 2.0 Authorization Framework  
- RFC 6750 — OAuth 2.0 Bearer Token Usage  
- RFC 8414 — OAuth 2.0 Authorization Server Metadata  
- NIST SP 800-207 — Zero Trust Architecture  
- Documentación oficial de **Keycloak**  
- Documentación de **Microsoft Active Directory**
- RFC 4120 — Kerberos Network Authentication Service
- OWASP Authentication Cheat Sheet
- FIDO Alliance — WebAuthn Specification
