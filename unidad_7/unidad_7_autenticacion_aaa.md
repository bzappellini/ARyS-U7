# 🧭 Unidad 7 — Servicios de Autenticación y Control de Acceso (AAA)

**Carrera:** Licenciatura en Sistemas  
**Materia:** Administración de Redes y Seguridad  
**Sede:** Trelew — UNPSJB  
**Duración total:** 3 horas (teórica)  

---

## 🎯 Objetivos de aprendizaje

- Comprender los principios del modelo AAA (Autenticación, Autorización y Auditoría).  
- Analizar los mecanismos de autenticación tradicionales y modernos.  
- Comprender los modelos de control de acceso (RBAC, ABAC, ACL).  
- Conocer los servicios de directorio, federación e identidad más utilizados.  

---

## 🧩 Contenidos teóricos (3 horas)

### 1. Introducción al modelo AAA
- Definición de **Autenticación**, **Autorización** y **Auditoría**.  
- Ejemplos: acceso a una red WiFi, VPN o sistema corporativo.  
- Relación entre los tres componentes en la seguridad de red.

> 💡 **Ejemplo:**  
> En una VPN:  
> - Autenticación: el usuario se valida con sus credenciales.  
> - Autorización: se define qué redes puede acceder.  
> - Auditoría: se registra cuánto tiempo y desde dónde se conectó.

---

### 2. Servicios de autenticación y directorios

#### LDAP (Lightweight Directory Access Protocol)
- Servicio de directorio jerárquico.  
- Estructura: DN, CN, OU, atributos.  
- Operaciones: *bind*, *search*, *modify*.  
- Uso en entornos empresariales para centralizar usuarios.

#### RADIUS
- Arquitectura cliente-servidor.  
- Usos típicos: autenticación WiFi, VPN y equipos de red.  
- Componentes: NAS (Network Access Server), servidor RADIUS.  
- Diferencias con TACACS+.  

#### Active Directory (AD)
- Combina LDAP + Kerberos.  
- Funciona como controlador de dominio.  
- Define políticas de grupo (GPO) y delegación de permisos.  

---

### 3. Autenticación basada en tickets — Kerberos
- Componentes principales: **AS**, **TGS**, **KDC**.  
- Flujo básico de autenticación:  
  1. El cliente solicita un *Ticket-Granting Ticket (TGT)*.  
  2. El KDC emite el ticket firmado.  
  3. El cliente obtiene tickets de servicio (TGS).  
- Integración con Active Directory.  
- Limitaciones: sincronización de tiempo, tickets caducos.

---

### 4. Autenticación moderna basada en tokens

#### JWT (JSON Web Token)
- Formato ligero para representar identidades.  
- Estructura: **Header**, **Payload**, **Signature**.  
- Firmas HMAC o RSA.  
- Claims, expiración, validación y revocación.

#### OAuth 2.0
- Framework para autorización delegada.  
- Roles: *Resource Owner*, *Client*, *Authorization Server*, *Resource Server*.  
- Flujos: *Authorization Code*, *Client Credentials*, *Device Flow*.

#### OpenID Connect (OIDC)
- Extiende OAuth 2.0 para autenticación de usuarios.  
- Tokens: *Access Token*, *ID Token*, *Refresh Token*.  
- Integración con Keycloak, Google, Azure AD, GitHub.

---

### 5. Federación de identidades y SSO
- Concepto de **Single Sign-On (SSO)**.  
- Federaciones entre dominios o empresas.  
- Protocolos:  
  - **SAML 2.0**  
  - **OpenID Connect (OIDC)**  
  - **WS-Federation**  
- Ejemplo: autenticación unificada con Keycloak y servicios web.

---

### 6. Autenticación multifactor (MFA) y tendencias actuales
- Factores de autenticación: algo que sabés, tenés o sos.  
- MFA / 2FA: TOTP, SMS, aplicaciones autenticadoras.  
- **FIDO2 / WebAuthn / Passkey:** autenticación sin contraseñas.  
- Concepto de **Zero Trust Authentication**.

---

### 7. Control de acceso
- **ACL (Access Control List):** permisos por recurso.  
- **RBAC (Role-Based Access Control):** permisos por rol.  
- **ABAC (Attribute-Based Access Control):** decisiones basadas en contexto o atributos.  
- Aplicación de políticas de acceso dinámico.

---

## 🔗 Conexiones y contexto

| Concepto | Tecnologías Relacionadas | Aplicación |
|-----------|--------------------------|-------------|
| **Autenticación** | Kerberos, OAuth2, OIDC, JWT, AD | Validación de identidad |
| **Autorización** | RBAC, ABAC, ACL | Control de acceso a recursos |
| **Auditoría** | Syslog, RADIUS accounting, AD logs | Registro y trazabilidad |

---

## 🧠 Evaluación sugerida

- **Cuestionario en Moodle:** preguntas teóricas sobre AAA, LDAP, OAuth y Kerberos.  
- **Pregunta reflexiva:**  
  > ¿Qué ventajas ofrece un esquema federado de identidad frente a un sistema de autenticación local?

---

## 📚 Recursos adicionales

- RFC 4511 — Lightweight Directory Access Protocol (LDAP)  
- RFC 6749 — OAuth 2.0 Authorization Framework  
- RFC 6750 — OAuth 2.0 Bearer Token Usage  
- RFC 8414 — OAuth 2.0 Authorization Server Metadata  
- NIST SP 800-207 — Zero Trust Architecture  
- Documentación oficial de **Keycloak**  
- Documentación de **Microsoft Active Directory**

---

## 💬 Cierre de la unidad
Esta unidad integra los tres pilares de la seguridad de identidad: autenticación, autorización y auditoría.  
Los alumnos deben poder explicar cómo se relacionan Kerberos, LDAP, OAuth y los modelos de control de acceso, y aplicar estos conocimientos tanto en entornos corporativos como en servicios web modernos.

