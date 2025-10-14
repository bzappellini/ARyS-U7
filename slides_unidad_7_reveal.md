<!-- Reveal.js Markdown deck for Unidad 7 (Teórica) -->

# Unidad 7 — AAA: Autenticación y Control de Acceso

**Administración de Redes y Seguridad — UNPSJB (Trelew)**  
**Duración:** 3 h (teórica)  
**Docente:** Bruno Zappellini

Note:
- Presentación breve del propósito de la unidad.
- Alinear expectativas: autenticación, federación y control de acceso.

---

## Agenda

1. Modelo AAA  
2. Servicios de autenticación y directorios (LDAP, RADIUS, AD)  
3. Kerberos (tickets)  
4. Tokens modernos: JWT, OAuth 2.0, OpenID Connect  
5. Federación y SSO  
6. MFA, FIDO2/Passkeys y Zero Trust  
7. Control de acceso: ACL, RBAC, ABAC

Note:
- Marcar tiempos aproximados por sección.

---

## 1) Modelo AAA

- **Autenticación**: probar identidad  
- **Autorización**: qué puede hacer  
- **Auditoría**: qué hizo (trazabilidad)

> Ejemplo VPN: login (A), perfiles de red (A), logs de sesión (A)

Note:
- Señalar que AAA se implementa en múltiples capas (app, red, SO).

---

## Principios clave

- Menor privilegio  
- *Defense-in-Depth*  
- *Separation of Duties*  
- *Zero Trust*: nunca confíes por defecto, valida siempre

Note:
- Conectar con políticas y cumplimiento (NIST, ISO 27001).

---

## 2) Servicios de autenticación y directorios

### LDAP — Directorio

- Árbol jerárquico (DN, OU, CN)  
- Operaciones: bind, search, modify  
- Centraliza usuarios/grupos, apps consultan

Note:
- Ejemplificar un DN y un filtro de búsqueda simple.

----

### RADIUS — Acceso de red

- Cliente/servidor (NAS ↔ RADIUS)  
- WiFi, VPN, switches/firewalls  
- Autenticación + *accounting* (auditoría)

> Diferencia vs TACACS+: granularidad de autorización, comandos, Cisco

Note:
- Mostrar flujo: supplicant → NAS → RADIUS → directorio/DB.

----

### Active Directory (AD)

- AD = LDAP + Kerberos (dominio)  
- GPO para políticas  
- Integración con endpoints y servidores

Note:
- Relacionar AD con Kerberos (autenticación por tickets).

---

## 3) Kerberos — Autenticación por tickets

**Componentes:** AS, TGS, KDC  
**Flujo (resumen):**
1. Cliente solicita TGT al AS  
2. KDC emite TGT  
3. Cliente pide ticket de servicio al TGS  
4. Acceso al servicio con ticket

> Requiere sincronización horaria precisa

Note:
- Mencionar *replay protection* y expiración de tickets.

---

## Kerberos en AD

- Login en dominio = obtención de TGT  
- *Single Sign-On* dentro del dominio  
- Delegación y *service principal names* (SPN)

Note:
- Casos: SMB, IIS, SQL Server; klist para ver tickets.

---

## 4) Tokens modernos

### JWT — JSON Web Token

- Estructura: **header.payload.signature** (Base64URL)  
- *Claims* estándar (iss, sub, aud, exp) y personalizados  
- Firmas HMAC/RSA/EC; validación y expiración

> Riesgos: almacenar secretos, *token leakage*, *audience/issuer* incorrectos

Note:
- Aclarar que JWT ≠ cifrado (a menos que JWE), por defecto es firmado.

----

### OAuth 2.0 — Autorización delegada

- Roles: Resource Owner, Client, AuthZ Server, Resource Server  
- Flujos:
  - **Authorization Code** (con PKCE en apps públicas)  
  - **Client Credentials** (servicio↔servicio)  
  - Device Flow (TV/IoT)

> *Access Token* (bearer), *Refresh Token*

Note:
- Recalcar PKCE y prácticas seguras en front-end/mobile.

----

### OpenID Connect — Identidad sobre OAuth2

- ID Token (JWT) con identidad verificada  
- *UserInfo endpoint*  
- *Discovery* y JWKS (metadatos del IdP)

> IdP comunes: Keycloak, Azure AD, Google, Okta

Note:
- Diferenciar *autenticación* (OIDC) de *autorización* (OAuth2).

---

## 5) Federación y SSO

- **SSO**: una sesión, múltiples aplicaciones  
- **Federación**: confianza entre dominios/organizaciones  
- Protocolos: **SAML 2.0**, **OIDC**, **WS-Fed**

> Trust mediante metadatos, certificados y endpoints bien definidos

Note:
- Ejemplo: IdP central (Keycloak) federando con apps SaaS.

---

## 6) MFA, FIDO2 y Zero Trust

- MFA/2FA: TOTP, push, U2F  
- **FIDO2/WebAuthn/Passkeys**: *phishing-resistant*  
- Políticas adaptativas: riesgo, geolocalización, dispositivo  
- **Zero Trust**: verificación continua + microsegmentación

Note:
- Ventajas de Passkeys vs SMS/OTP.

---

## 7) Control de acceso

### ACL — Listas de Control de Acceso

- Permisos por recurso/usuario  
- Escalable en objetos simples, complejo a gran escala

----

### RBAC — Basado en Roles

- Permisos agrupados por rol (admin, auditor, lector)  
- Facilita alta/rotación de personal

----

### ABAC — Basado en Atributos

- Decisiones por atributos (usuario, recurso, contexto)  
- Políticas dinámicas (horario, ubicación, nivel de riesgo)

> Motores de políticas (XACML, OPA/Rego)

Note:
- Contrastar RBAC vs ABAC y uso combinado.

---

## Diseño de una arquitectura de identidad

- IdP central (OIDC/SAML)  
- Integración con AD/LDAP  
- MFA obligatorio para riesgo/roles críticos  
- Autorización por RBAC + condiciones ABAC  
- Auditoría y *monitoring* (SIEM)

Note:
- Presentar *blueprint* de referencia aplicable a PJ o Universidad.

---

## Buenas prácticas y anti‑patrones

- No reinventar login; usar IdP y estándares  
- Minimizar tiempo de vida de tokens; *ROTATE* secretos  
- PKCE en clientes públicos; *https-only*  
- *Scopes* mínimos; *claims* necesarios  
- *Logs* firmados y retenidos; alertas de anomalías

Note:
- Ejemplos de incidentes por manejo inseguro de tokens.

---

## Cierre y próximos pasos

- Repaso: AAA → Kerberos → OAuth/OIDC → Federación → MFA → Control de acceso  
- Avanzar a prácticas: Keycloak, flujos OIDC y RBAC/ABAC  
- Lecturas y normas de referencia

Note:
- Dejar tiempo para preguntas.

---

## Bibliografía / Referencias

- RFC 4511 (LDAP)  
- RFC 6749 (OAuth 2.0), RFC 6750 (Bearer), RFC 8414 (Metadata)  
- SAML 2.0 Core  
- WebAuthn / FIDO2  
- NIST SP 800‑63 (IA), NIST SP 800‑207 (Zero Trust)  
- Documentación oficial: Keycloak, Azure AD, Google Identity

Note:
- Indicar enlaces en el campus virtual.

