# OAuth 2.0 y OpenID Connect

## Introducción

OAuth 2.0 y OpenID Connect (OIDC) son los estándares modernos para autorización y autenticación en aplicaciones web, móviles y APIs.

### Diferencias Clave

| Aspecto | OAuth 2.0 | OpenID Connect |
|---------|-----------|----------------|
| **Propósito** | Autorización | Autenticación + Autorización |
| **Pregunta** | "¿Qué puede hacer?" | "¿Quién eres?" |
| **Token principal** | Access Token | ID Token (JWT) |
| **Información** | Scopes/permisos | Identidad del usuario |
| **Estándar** | RFC 6749 | Construido sobre OAuth 2.0 |

---

## OAuth 2.0

OAuth 2.0 es un framework de autorización que permite a aplicaciones obtener acceso limitado a recursos de un usuario sin exponer sus credenciales.

### Roles

**1. Resource Owner (Dueño del recurso)**
- Usuario que posee los datos
- Autoriza acceso a sus recursos

**2. Client (Cliente)**
- Aplicación que solicita acceso
- Puede ser web, móvil, SPA, backend

**3. Authorization Server**
- Emite tokens después de autenticar al usuario
- Ejemplos: Keycloak, Auth0, Okta

**4. Resource Server**
- API que protege recursos
- Valida tokens antes de dar acceso

### Flujos de Autorización

#### 1. Authorization Code Flow ⭐⭐⭐⭐⭐

**Uso**: Aplicaciones web con backend seguro

**Características**:
- Más seguro
- Token nunca expuesto al navegador
- Requiere client_secret

**Flujo**:
```
1. Usuario → Client: Click "Login"
2. Client → Authorization Server: Redirect a /authorize
3. Usuario autentica en Authorization Server
4. Authorization Server → Client: Code via redirect
5. Client → Authorization Server: Exchange code + secret → tokens
6. Client recibe: access_token, refresh_token, id_token
```

**Ejemplo de solicitud de código**:
```http
GET /authorize?
  response_type=code&
  client_id=app123&
  redirect_uri=https://app.com/callback&
  scope=openid profile email&
  state=abc123
```

**Ejemplo de intercambio**:
```http
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=AUTH_CODE&
redirect_uri=https://app.com/callback&
client_id=app123&
client_secret=SECRET
```

#### 2. Authorization Code Flow + PKCE ⭐⭐⭐⭐⭐

**Uso**: Aplicaciones móviles, SPAs (Single Page Apps)

**PKCE**: Proof Key for Code Exchange

**Características**:
- No requiere client_secret
- Protege contra ataques de intercepción de código
- Estándar para clientes públicos

**Parámetros adicionales**:
- `code_challenge`: Hash del code_verifier
- `code_challenge_method`: S256 (SHA256)
- `code_verifier`: String aleatorio al intercambiar

**Flujo**:
```
1. Client genera code_verifier aleatorio
2. Client calcula code_challenge = SHA256(code_verifier)
3. Client solicita code con code_challenge
4. Authorization Server guarda code_challenge
5. Client intercambia code + code_verifier
6. Server valida: SHA256(code_verifier) == code_challenge
```

**Ejemplo**:
```bash
# Generar code_verifier
CODE_VERIFIER=$(openssl rand -base64 32 | tr -d "=+/" | cut -c1-43)

# Calcular code_challenge
CODE_CHALLENGE=$(echo -n $CODE_VERIFIER | \
  openssl dgst -binary -sha256 | \
  base64 | tr -d "=+/" | cut -c1-43)

# Solicitar código
# GET /authorize?...&code_challenge=$CODE_CHALLENGE&code_challenge_method=S256

# Intercambiar código
# POST /token
# grant_type=authorization_code&code=...&code_verifier=$CODE_VERIFIER
```

#### 3. Client Credentials Flow ⭐⭐⭐⭐

**Uso**: Comunicación entre servicios (M2M)

**Características**:
- No hay usuario humano
- Cliente se autentica con id + secret
- Solo devuelve access_token

**Flujo**:
```
1. Service A → Authorization Server: client_id + client_secret
2. Authorization Server valida credenciales
3. Authorization Server → Service A: access_token
4. Service A → Service B API: access_token
```

**Ejemplo**:
```http
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&
client_id=service-a&
client_secret=SECRET&
scope=api:read api:write
```

#### 4. Resource Owner Password Credentials ⭐ (NO RECOMENDADO)

**Uso**: Solo cuando no hay alternativa (legacy)

**Problemas**:
- ❌ Cliente tiene acceso a la contraseña
- ❌ No hay experiencia de login centralizada
- ❌ Dificulta MFA
- ❌ No cumple con mejores prácticas

**Ejemplo** (solo para entender):
```http
POST /token

grant_type=password&
username=usuario&
password=contraseña&
client_id=app&
client_secret=secret
```

#### 5. Implicit Flow ❌ (DEPRECADO)

**Estado**: Deprecado por motivos de seguridad

**Por qué NO usar**:
- Token expuesto en URL
- Sin refresh token
- Vulnerable a ataques XSS
- Reemplazar con Authorization Code + PKCE

### Tokens en OAuth 2.0

#### Access Token

**Propósito**: Acceder a APIs protegidas

**Formato**: Opaco o JWT

**Características**:
- Corto tiempo de vida (5-60 minutos)
- Bearer token: "Bearer XXXXX"
- Incluye scopes autorizados

**Ejemplo de uso**:
```http
GET /api/users
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...
```

#### Refresh Token

**Propósito**: Obtener nuevos access tokens sin reautenticar

**Características**:
- Largo tiempo de vida (días/semanas)
- Solo usado con Authorization Server
- Puede ser revocado

**Uso**:
```http
POST /token

grant_type=refresh_token&
refresh_token=REFRESH_TOKEN&
client_id=app&
client_secret=secret
```

### Scopes

Los **scopes** definen los permisos que solicita una aplicación.

**Scopes comunes**:
- `openid`: Obligatorio para OIDC
- `profile`: Información de perfil (nombre, foto)
- `email`: Dirección de email
- `address`: Dirección física
- `phone`: Número de teléfono

**Scopes personalizados**:
- `read:users`
- `write:posts`
- `admin:all`

**Ejemplo de solicitud**:
```
scope=openid profile email read:users write:posts
```

---

## OpenID Connect (OIDC)

OpenID Connect extiende OAuth 2.0 para proveer autenticación y obtener información de identidad del usuario.

### ID Token (JWT)

**Propósito**: Confirmar la identidad del usuario

**Formato**: Siempre JWT (JSON Web Token)

**Estructura**:
```
Header.Payload.Signature
```

**Claims estándar**:
```json
{
  "iss": "https://auth.ejemplo.com",        // Issuer
  "sub": "123456789",                        // Subject (user ID)
  "aud": "app-client-id",                    // Audience
  "exp": 1234567890,                         // Expiration time
  "iat": 1234567890,                         // Issued at
  "auth_time": 1234567890,                   // Authentication time
  "nonce": "abc123",                         // Replay protection
  "acr": "1",                                // Authentication context
  "amr": ["pwd", "mfa"],                     // Authentication methods
  
  // Profile claims
  "name": "Bruno Zappellini",
  "given_name": "Bruno",
  "family_name": "Zappellini",
  "preferred_username": "bzappellini",
  "email": "bruno@unpsjb.edu.ar",
  "email_verified": true,
  "picture": "https://...",
  "locale": "es-AR"
}
```

### UserInfo Endpoint

Endpoint para obtener información adicional del usuario autenticado.

**Uso**:
```http
GET /userinfo
Authorization: Bearer ACCESS_TOKEN
```

**Respuesta**:
```json
{
  "sub": "123456789",
  "name": "Bruno Zappellini",
  "email": "bruno@unpsjb.edu.ar",
  "email_verified": true,
  "roles": ["admin", "profesor"]
}
```

### Discovery y Metadatos

OIDC provee un endpoint de descubrimiento para configuración automática.

**URL**:
```
https://auth.ejemplo.com/.well-known/openid-configuration
```

**Información incluida**:
```json
{
  "issuer": "https://auth.ejemplo.com",
  "authorization_endpoint": "https://auth.ejemplo.com/authorize",
  "token_endpoint": "https://auth.ejemplo.com/token",
  "userinfo_endpoint": "https://auth.ejemplo.com/userinfo",
  "jwks_uri": "https://auth.ejemplo.com/.well-known/jwks.json",
  "response_types_supported": ["code", "token", "id_token"],
  "grant_types_supported": ["authorization_code", "refresh_token"],
  "scopes_supported": ["openid", "profile", "email"],
  "claims_supported": ["sub", "name", "email", "..."]
}
```

### JWKS (JSON Web Key Set)

Conjunto de claves públicas para validar la firma de los JWT.

**Endpoint**:
```
https://auth.ejemplo.com/.well-known/jwks.json
```

**Ejemplo**:
```json
{
  "keys": [
    {
      "kty": "RSA",
      "use": "sig",
      "kid": "key-id-1",
      "n": "0vx7agoebGcQSuuPiLJXZptN9nndrQmbXEps2aiAFbWhM78LhWx...",
      "e": "AQAB"
    }
  ]
}
```

---

## JSON Web Tokens (JWT)

### Estructura

**Formato**:
```
HEADER.PAYLOAD.SIGNATURE
```

**Header**:
```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "key-id"
}
```

**Payload (Claims)**:
```json
{
  "sub": "user-id",
  "iat": 1234567890,
  "exp": 1234567990,
  "custom_claim": "value"
}
```

**Signature**:
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

### Tipos de Claims

**Registered Claims (Estándar)**:
- `iss` (issuer): Quien emitió el token
- `sub` (subject): Identificador del usuario
- `aud` (audience): Para quién es el token
- `exp` (expiration): Cuándo expira
- `nbf` (not before): Válido desde
- `iat` (issued at): Cuándo se emitió
- `jti` (JWT ID): Identificador único del token

**Public Claims**: Definidos en registro IANA
**Private Claims**: Personalizados

### Validación de JWT

**Pasos**:
1. ✅ Verificar firma con clave pública (JWKS)
2. ✅ Validar `exp` (no expirado)
3. ✅ Validar `iss` (issuer esperado)
4. ✅ Validar `aud` (audience correcto)
5. ✅ Validar `nbf` si está presente
6. ✅ Validar algoritmo (`alg` en whitelist)

**Ejemplo en Node.js**:
```javascript
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const client = jwksClient({
  jwksUri: 'https://auth.ejemplo.com/.well-known/jwks.json'
});

function getKey(header, callback) {
  client.getSigningKey(header.kid, (err, key) => {
    const signingKey = key.publicKey || key.rsaPublicKey;
    callback(null, signingKey);
  });
}

jwt.verify(token, getKey, {
  audience: 'app-client-id',
  issuer: 'https://auth.ejemplo.com',
  algorithms: ['RS256']
}, (err, decoded) => {
  if (err) {
    console.error('Token inválido');
  } else {
    console.log('Token válido:', decoded);
  }
});
```

### Seguridad de JWT

**❌ Malas prácticas**:
- Almacenar datos sensibles en payload (es solo Base64)
- Usar algoritmo `none`
- No validar firma
- Ignorar expiración
- Compartir secret en cliente

**✅ Buenas prácticas**:
- Usar RS256 o ES256 (asimétrico)
- Tiempo de vida corto (5-15 minutos)
- Validar todos los claims
- Usar HTTPS siempre
- Implementar lista negra para revocación
- Rotar claves regularmente

---

## Implementaciones y Providers

### Identity Providers Populares

**Open Source**:
- **Keycloak**: RedHat, muy completo, self-hosted
- **Ory Hydra**: Moderno, cloud-native
- **Gluu**: Empresarial, cumplimiento normativo
- **IdentityServer**: .NET, enterprise

**SaaS/Cloud**:
- **Auth0**: Developer-friendly, Okta
- **Okta**: Empresarial, completo
- **Azure AD** (Entra ID): Microsoft, integrado con 365
- **Google Identity**: Consumer y workspace
- **AWS Cognito**: Integrado con AWS
- **Firebase Auth**: Google, mobile-first

### Keycloak

Solución open-source completa de IAM (Identity and Access Management).

**Características**:
- OIDC, OAuth 2.0, SAML 2.0
- User Federation (LDAP, AD)
- Social Login (Google, Facebook, GitHub)
- MFA/2FA integrado
- Administración de usuarios
- Temas personalizables
- Cliente admin REST API

**Instalación rápida**:
```bash
docker run -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:latest \
  start-dev
```

---

## Single Sign-On (SSO)

### Concepto

Un usuario se autentica una vez y accede a múltiples aplicaciones sin volver a ingresar credenciales.

### Flujo SSO con OIDC

```
1. Usuario → App A: Click "Login"
2. App A → IdP: Redirect a /authorize
3. Usuario NO tiene sesión en IdP → Login
4. IdP crea sesión (cookie)
5. IdP → App A: tokens
6. Usuario autenticado en App A

Luego...

7. Usuario → App B: Click "Login"
8. App B → IdP: Redirect a /authorize
9. Usuario YA tiene sesión en IdP → Skip login
10. IdP → App B: tokens
11. Usuario autenticado en App B (¡sin login!)
```

### Single Logout (SLO)

Cerrar sesión en una app cierra sesión en todas.

**Métodos**:
- **Front-Channel**: Iframes invisibles
- **Back-Channel**: Notificaciones HTTP
- **RP-Initiated**: Cliente solicita logout

---

## Mejores Prácticas

### Seguridad

✅ **Usar HTTPS siempre**
✅ **Implementar PKCE para SPAs y móviles**
✅ **Validar state parameter** (CSRF protection)
✅ **Usar nonce en ID tokens** (replay protection)
✅ **Tokens de corta duración**
✅ **Refresh token rotation**
✅ **Whitelist de redirect_uris**
✅ **Validar audience y issuer**

### Desarrollo

✅ **Usar bibliotecas establecidas** (no implementar desde cero)
✅ **Manejar errores gracefully**
✅ **Implementar timeout en tokens**
✅ **Logs de autenticación**
✅ **Testing exhaustivo de flujos**

### Producción

✅ **Monitoreo de autenticaciones**
✅ **Alertas de comportamientos anómalos**
✅ **Backup de configuración**
✅ **Documentación de integraciones**
✅ **Plan de recuperación de desastres**

---

## Recursos

### Especificaciones
- [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [RFC 6750 - Bearer Token](https://tools.ietf.org/html/rfc6750)
- [RFC 7636 - PKCE](https://tools.ietf.org/html/rfc7636)
- [OpenID Connect Core](https://openid.net/specs/openid-connect-core-1_0.html)
- [RFC 7519 - JWT](https://tools.ietf.org/html/rfc7519)

### Herramientas
- [jwt.io](https://jwt.io) - Decodificar y validar JWT
- [OAuth Debugger](https://oauthdebugger.com) - Probar flujos OAuth
- [Postman](https://www.postman.com) - Testing de APIs

### Documentación
- [Keycloak Docs](https://www.keycloak.org/documentation)
- [Auth0 Docs](https://auth0.com/docs)
- [OAuth 2.0 Simplified](https://www.oauth.com)
