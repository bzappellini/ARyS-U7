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

Kerberos evita enviar contraseñas por la red.  
En su lugar, usa **tickets cifrados** emitidos por un servidor central (**KDC**).

### 🔹 Componentes
- **AS (Authentication Server):** autentica al usuario.  
- **TGS (Ticket Granting Server):** emite tickets de servicio.  
- **KDC (Key Distribution Center):** combina AS + TGS.  
- **Cliente / Servidor:** quien solicita y quien ofrece el servicio.

### 🔹 Flujo simplificado

1. Usuario pide un **Ticket Granting Ticket (TGT)** al AS.  
2. AS verifica las credenciales y envía un TGT cifrado con la clave del TGS.  
3. El usuario usa el TGT para obtener tickets de servicio (TGS).  
4. Los servicios validan el ticket sin reenviar contraseñas.

💡 **Ejemplo:**  
Cuando un usuario inicia sesión en un dominio Windows, obtiene un TGT válido por un tiempo determinado.  
Ese ticket se usa para acceder automáticamente a recursos como servidores de archivos o correo sin volver a escribir la contraseña.

🧠 **Ventajas:**
- Evita el envío de contraseñas.  
- Soporta SSO (Single Sign-On).  
- Permite autenticación mutua (cliente ↔ servidor).

🔒 **Requisito crítico:** sincronización horaria precisa (NTP).

---

## 🧩 4. Autenticación moderna basada en tokens

Con el auge de la web y la nube, se abandonaron los “tickets” de red local en favor de **tokens portátiles y estandarizados** (JWT, OAuth2, OIDC).

---

### 🔹 JWT (JSON Web Token)

- Formato estándar para representar identidades en JSON.  
- Firmado (no cifrado) con HMAC o RSA.  
- Se usa para autenticar en APIs y sistemas web.

**Estructura:**
```
HEADER.PAYLOAD.SIGNATURE
```

Ejemplo (simplificado):
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "sub": "bruno",
  "role": "admin",
  "exp": 1732130400
}
```

🧠 **Concepto clave:**  
Un JWT se firma para garantizar integridad, pero **no es confidencial** (cualquiera puede leerlo si tiene el token).

💡 **Ejemplo:**  
En una API REST, el backend valida la firma del JWT para verificar que proviene de un servidor confiable (como Keycloak o Auth0).

---

### 🔹 OAuth 2.0 — Autorización delegada

Protocolo que permite a una aplicación acceder a recursos en nombre de un usuario, **sin conocer su contraseña**.

**Roles:**
- **Resource Owner:** el usuario.  
- **Client:** la aplicación que quiere acceso.  
- **Authorization Server:** quien valida y emite tokens.  
- **Resource Server:** la API que protege los datos.

**Flujo clásico (Authorization Code Flow):**
1. La app redirige al usuario al servidor de autorización.  
2. El usuario concede acceso.  
3. La app recibe un código y lo intercambia por un *Access Token*.

💡 **Ejemplo:**  
“Iniciar sesión con Google” usa OAuth 2.0.  
Tu contraseña nunca llega a la app: Google valida tu identidad y emite un token temporal.

---

### 🔹 OpenID Connect (OIDC)

Extiende OAuth 2.0 para incluir **autenticación** además de autorización.

**Tokens principales:**
- **Access Token:** permite acceder a APIs.  
- **ID Token:** contiene identidad del usuario (nombre, email, roles).  
- **Refresh Token:** permite renovar el Access Token.

🧠 **Concepto clave:**  
OAuth 2.0 autoriza acceso → OIDC autentica identidades.  
Ambos pueden coexistir en una misma arquitectura.

💡 **Ejemplo:**  
Un usuario inicia sesión en Keycloak, que actúa como *Identity Provider* (IdP) y emite un **ID Token** JWT que identifica al usuario en múltiples aplicaciones.

---

## 🧩 5. Federación de identidades y Single Sign-On (SSO)

La **federación de identidades** permite compartir autenticación entre dominios u organizaciones.

**Protocolos principales:**
- **SAML 2.0** (XML, usado en entornos corporativos).  
- **OpenID Connect** (JSON, usado en web y cloud).  
- **WS-Federation** (Microsoft).

💡 **Ejemplo:**
- El sistema de la UNPSJB confía en el IdP del Poder Judicial para autenticar a un usuario (federación entre dominios distintos).  
- Una vez autenticado, accede a Moodle, correo y portal interno sin volver a loguearse (**SSO**).

🧠 **Beneficios:**  
- Menos contraseñas.  
- Menor superficie de ataque.  
- Experiencia unificada de acceso.

⚠️ **Riesgos:**  
- Un fallo en el IdP afecta a todas las aplicaciones federadas.  
- Requiere gestión cuidadosa de certificados y confianza entre dominios.

---

## 🧩 6. Autenticación multifactor (MFA) y Zero Trust

### 🔹 MFA / 2FA
Combina múltiples factores para autenticar a un usuario:

| Tipo | Ejemplo |
|------|----------|
| Algo que sabés | Contraseña |
| Algo que tenés | Token físico, app móvil |
| Algo que sos | Huella digital, rostro |

**Ejemplo práctico:**
- Login con contraseña + código TOTP (Google Authenticator o Keycloak MFA).  
- Seguridad elevada frente al phishing o robo de credenciales.

---

### 🔹 FIDO2 / Passkey
- Nuevo estándar de autenticación sin contraseña.  
- Basado en criptografía asimétrica (clave pública / privada).  
- Compatible con Windows Hello, Android, iOS, etc.  
- Previene ataques de phishing y *credential stuffing*.

---

### 🔹 Zero Trust Authentication
- Modelo de seguridad que **no confía en nadie por defecto**.  
- Cada acceso debe verificarse continuamente, incluso dentro de la red interna.  
- Aplica autenticación adaptativa (por riesgo, ubicación, dispositivo, hora).

🧠 **Resumen conceptual:**
> “Nunca confíes, verifica siempre.”

---

## 🧩 7. Control de acceso: ACL, RBAC, ABAC

### 🔹 ACL (Access Control List)
- Permisos aplicados a cada recurso (archivo, carpeta, endpoint).  
- Ejemplo: permisos UNIX (`rwx` para usuario, grupo, otros).

### 🔹 RBAC (Role-Based Access Control)
- Define roles que agrupan permisos.  
- Ejemplo:  
  - Rol *admin*: puede crear y borrar usuarios.  
  - Rol *docente*: puede editar calificaciones.  
  - Rol *alumno*: solo lectura.

### 🔹 ABAC (Attribute-Based Access Control)
- Usa atributos del usuario, recurso o entorno.  
- Ejemplo: permitir acceso si `departamento == "Redes"` y `ubicación == "Argentina"`.

💡 **Aplicación práctica:**
- En Keycloak o Azure AD, se combinan RBAC + ABAC + MFA bajo un modelo **Zero Trust**.

---

## 🧩 Conclusiones

- AAA estructura la seguridad de identidades.  
- Kerberos fue el punto de partida de los modelos modernos.  
- OAuth 2.0 y OpenID Connect lideran la autenticación en la nube.  
- MFA y Zero Trust redefinen la seguridad en entornos distribuidos.  
- RBAC y ABAC completan el ciclo de control de acceso.  

---

## 📚 Referencias y lecturas recomendadas

- RFC 4511 — LDAP  
- RFC 4120 — Kerberos V5  
- RFC 6749 — OAuth 2.0  
- RFC 8414 — OAuth 2.0 Server Metadata  
- NIST SP 800-63 — Digital Identity Guidelines  
- NIST SP 800-207 — Zero Trust Architecture  
- Documentación de **Keycloak**, **Azure AD**, **Google Identity**  
- OWASP Cheat Sheets: Authentication & Session Management  

---

✍️ **Actividad sugerida:**  
Analizar cómo se implementa la autenticación y autorización en un sistema real (por ejemplo, Keycloak o GitHub) e identificar qué protocolos y modelos utiliza (OAuth, OIDC, RBAC, MFA, etc.).

