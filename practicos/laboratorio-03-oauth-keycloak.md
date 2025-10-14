# 🧪 Laboratorio 3: OAuth 2.0 / OIDC con Keycloak

## Información General

- **Dificultad**: Intermedia
- **Duración estimada**: 90 minutos
- **Prerrequisitos**: 
  - Laboratorios 1 y 2 completados
  - Conocimientos de HTTP/REST
  - Comprensión de JWT

## Objetivos

Al completar este laboratorio, serás capaz de:

1. Instalar y configurar Keycloak
2. Crear realms, clientes y usuarios
3. Implementar flujos de autorización OAuth 2.0
4. Obtener y validar tokens JWT
5. Diferenciar entre Access Token e ID Token
6. Comprender los diferentes flujos de OAuth 2.0

---

## Parte 1: Instalación y Configuración de Keycloak (20 min)

### Paso 1.1: Instalación con Docker

```bash
# Descargar y ejecutar Keycloak
docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:latest \
  start-dev

# Verificar que está corriendo
docker logs -f keycloak
```

Esperar a que aparezca el mensaje:
```
Keycloak ... started in ...ms
```

### Paso 1.2: Acceder a la consola de administración

1. Abrir navegador en: `http://localhost:8080`
2. Acceder a "Administration Console"
3. Login:
   - Usuario: `admin`
   - Contraseña: `admin`

---

## Parte 2: Configuración Básica (20 min)

### Paso 2.1: Crear un Realm

Un **realm** es un espacio aislado para gestionar usuarios, aplicaciones y configuraciones.

1. En el menú superior izquierdo, hacer clic en el dropdown (actualmente "Master")
2. Click en "Create Realm"
3. Configuración:
   - **Realm name**: `arys-curso`
   - **Enabled**: ON
4. Click "Create"

### Paso 2.2: Crear un Cliente (Aplicación)

1. En el menú lateral, ir a **Clients**
2. Click "Create client"
3. **General Settings**:
   - **Client type**: OpenID Connect
   - **Client ID**: `app-practica`
4. Click "Next"
5. **Capability config**:
   - **Client authentication**: ON
   - **Authorization**: OFF
   - **Authentication flow**:
     - ☑ Standard flow
     - ☑ Direct access grants
6. Click "Next"
7. **Login settings**:
   - **Valid redirect URIs**: `http://localhost:3000/*`
   - **Web origins**: `http://localhost:3000`
8. Click "Save"

### Paso 2.3: Obtener credenciales del cliente

1. En la configuración del cliente, ir a la pestaña **Credentials**
2. Copiar el **Client secret** (lo usaremos después)

### Paso 2.4: Crear usuarios

1. En el menú lateral, ir a **Users**
2. Click "Add user"
3. Configuración:
   - **Username**: `bruno`
   - **Email**: `bruno@unpsjb.edu.ar`
   - **First name**: `Bruno`
   - **Last name**: `Zappellini`
   - **Email verified**: ON
4. Click "Create"
5. Ir a la pestaña **Credentials**
6. Click "Set password"
7. Ingresar contraseña: `Password123`
8. **Temporary**: OFF
9. Click "Save"

Repetir para crear otro usuario:
- Username: `alumno1`
- Email: `alumno1@unpsjb.edu.ar`
- Password: `Password123`

---

## Parte 3: Authorization Code Flow (30 min)

### Objetivo

Implementar el flujo de autorización más seguro de OAuth 2.0.

### Paso 3.1: Configurar endpoints

Obtener configuración OIDC Discovery:

```bash
curl http://localhost:8080/realms/arys-curso/.well-known/openid-configuration | jq
```

Guardar los siguientes endpoints:
- `authorization_endpoint`
- `token_endpoint`
- `userinfo_endpoint`
- `jwks_uri`

### Paso 3.2: Flujo de autorización

**1. Solicitar código de autorización**

Construir URL (reemplazar `CLIENT_ID` con `app-practica`):

```
http://localhost:8080/realms/arys-curso/protocol/openid-connect/auth?
  client_id=app-practica&
  response_type=code&
  scope=openid profile email&
  redirect_uri=http://localhost:3000/callback&
  state=abc123
```

**Pasos:**
1. Abrir la URL en el navegador
2. Login con usuario `bruno` / `Password123`
3. Observar la redirección a `http://localhost:3000/callback?code=...`
4. Copiar el valor del parámetro `code`

**2. Intercambiar código por tokens**

```bash
# Reemplazar:
# - CLIENT_SECRET con el secret del paso 2.3
# - AUTHORIZATION_CODE con el code obtenido

curl -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "code=AUTHORIZATION_CODE" \
  -d "redirect_uri=http://localhost:3000/callback" | jq
```

Respuesta esperada:
```json
{
  "access_token": "eyJhbGc...",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJhbGc...",
  "token_type": "Bearer",
  "id_token": "eyJhbGc...",
  "not-before-policy": 0,
  "session_state": "...",
  "scope": "openid profile email"
}
```

### Paso 3.3: Analizar los tokens

**Decodificar ID Token:**

Copiar el `id_token` y decodificarlo en https://jwt.io

Observar los claims:
```json
{
  "exp": 1234567890,
  "iat": 1234567890,
  "auth_time": 1234567890,
  "jti": "...",
  "iss": "http://localhost:8080/realms/arys-curso",
  "aud": "app-practica",
  "sub": "...",
  "typ": "ID",
  "azp": "app-practica",
  "session_state": "...",
  "acr": "1",
  "email_verified": true,
  "name": "Bruno Zappellini",
  "preferred_username": "bruno",
  "given_name": "Bruno",
  "family_name": "Zappellini",
  "email": "bruno@unpsjb.edu.ar"
}
```

**Análisis:**
- `iss` (issuer): quién emitió el token
- `sub` (subject): identificador único del usuario
- `aud` (audience): para quién es el token
- `exp` (expiration): cuándo expira
- `iat` (issued at): cuándo se emitió

---

## Parte 4: Client Credentials Flow (15 min)

### Objetivo

Implementar autenticación entre servicios (machine-to-machine).

### Paso 4.1: Configurar cliente para M2M

1. Crear nuevo cliente:
   - **Client ID**: `servicio-backend`
   - **Client authentication**: ON
   - **Standard flow**: OFF
   - **Direct access grants**: OFF
   - **Service accounts roles**: ON
2. Guardar Client Secret

### Paso 4.2: Obtener Access Token

```bash
# Reemplazar CLIENT_SECRET con el secret del servicio

curl -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=servicio-backend" \
  -d "client_secret=CLIENT_SECRET" | jq
```

Respuesta:
```json
{
  "access_token": "eyJhbGc...",
  "expires_in": 300,
  "token_type": "Bearer",
  "scope": "profile email"
}
```

**Nota:** Este flujo NO devuelve `id_token` porque no hay un usuario humano involucrado.

---

## Parte 5: Validación de Tokens (20 min)

### Paso 5.1: Validar token en el Resource Server

```bash
# Guardar el access_token en una variable
ACCESS_TOKEN="eyJhbGc..."

# Llamar al endpoint de UserInfo
curl http://localhost:8080/realms/arys-curso/protocol/openid-connect/userinfo \
  -H "Authorization: Bearer $ACCESS_TOKEN" | jq
```

### Paso 5.2: Introspección de tokens

```bash
# Reemplazar CLIENT_SECRET y ACCESS_TOKEN

curl -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token/introspect \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "token=$ACCESS_TOKEN" | jq
```

Respuesta:
```json
{
  "exp": 1234567890,
  "iat": 1234567890,
  "jti": "...",
  "iss": "http://localhost:8080/realms/arys-curso",
  "aud": "account",
  "sub": "...",
  "typ": "Bearer",
  "azp": "app-practica",
  "session_state": "...",
  "acr": "1",
  "scope": "openid profile email",
  "email_verified": true,
  "name": "Bruno Zappellini",
  "preferred_username": "bruno",
  "given_name": "Bruno",
  "family_name": "Zappellini",
  "email": "bruno@unpsjb.edu.ar",
  "active": true
}
```

### Paso 5.3: Refresh Token

```bash
# Reemplazar CLIENT_SECRET y REFRESH_TOKEN

curl -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=refresh_token" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "refresh_token=REFRESH_TOKEN" | jq
```

---

## Ejercicios Adicionales

### Ejercicio 1: Resource Owner Password Credentials (NO RECOMENDADO en producción)

```bash
curl -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "username=bruno" \
  -d "password=Password123" | jq
```

**¿Por qué NO usar este flujo?**
- La aplicación tiene acceso directo a la contraseña del usuario
- No hay experiencia de login centralizada
- Dificulta implementar MFA

### Ejercicio 2: Logout

```bash
# Reemplazar CLIENT_SECRET y REFRESH_TOKEN

curl -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/logout \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "refresh_token=REFRESH_TOKEN"
```

---

## Análisis y Diferencias

### Access Token vs ID Token

| Característica | Access Token | ID Token |
|----------------|--------------|----------|
| **Propósito** | Acceder a APIs/recursos | Autenticar usuario |
| **Audiencia** | Resource Server | Cliente (app) |
| **Formato** | Opaco o JWT | Siempre JWT |
| **Claims** | Permisos, scopes | Identidad del usuario |
| **Validación** | Resource Server | Cliente |

### Flujos de OAuth 2.0

| Flujo | Uso | Seguridad |
|-------|-----|-----------|
| **Authorization Code** | Apps web con backend | ⭐⭐⭐⭐⭐ Más seguro |
| **Authorization Code + PKCE** | Apps móviles/SPA | ⭐⭐⭐⭐⭐ Más seguro |
| **Client Credentials** | Machine-to-Machine | ⭐⭐⭐⭐ Seguro |
| **Implicit** | (DEPRECADO) | ⭐⭐ No recomendado |
| **Password** | (LEGACY) | ⭐ Evitar |

---

## Verificación y Evaluación

### Checklist de Verificación

- [ ] Keycloak instalado y accesible
- [ ] Realm `arys-curso` creado
- [ ] Cliente `app-practica` configurado
- [ ] Al menos 2 usuarios creados
- [ ] Authorization Code Flow completado exitosamente
- [ ] Tokens obtenidos y decodificados
- [ ] ID Token y Access Token analizados
- [ ] Client Credentials Flow probado
- [ ] Introspección de tokens exitosa
- [ ] Refresh token probado

### Preguntas de Evaluación

1. **¿Cuál es la diferencia entre OAuth 2.0 y OpenID Connect?**

2. **¿Por qué el Authorization Code Flow es más seguro que el Implicit Flow?**

3. **¿Qué información contiene un ID Token y cuál es su propósito?**

4. **¿Cuándo usarías Client Credentials Flow vs Authorization Code Flow?**

5. **¿Qué es PKCE y por qué es importante para aplicaciones móviles y SPA?**

---

## Conclusiones

Al completar este laboratorio, has:

✅ Instalado y configurado Keycloak como Identity Provider  
✅ Implementado Authorization Code Flow  
✅ Trabajado con Client Credentials para M2M  
✅ Analizado y validado tokens JWT  
✅ Comprendido la diferencia entre Access Token e ID Token  
✅ Explorado diferentes flujos de OAuth 2.0

**Próximos pasos:**
- Laboratorio 4: Control de acceso con roles (RBAC)
- Integrar Keycloak con una aplicación web real
- Implementar SSO entre múltiples aplicaciones
