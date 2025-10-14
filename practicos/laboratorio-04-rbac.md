# 🧪 Laboratorio 4: Control de Acceso con Roles (RBAC)

## Información General

- **Dificultad**: Intermedia
- **Duración estimada**: 60 minutos
- **Prerrequisitos**: 
  - Laboratorio 3 completado
  - Keycloak configurado y corriendo
  - Comprensión de OAuth 2.0 y JWT

## Objetivos

Al completar este laboratorio, serás capaz de:

1. Definir roles en Keycloak
2. Asignar roles a usuarios
3. Configurar permisos basados en roles
4. Validar tokens con información de roles
5. Implementar control de acceso basado en roles en una API

---

## Parte 1: Configuración de Roles (20 min)

### Paso 1.1: Crear roles de realm

1. En Keycloak, asegurarse de estar en el realm `arys-curso`
2. Ir a **Realm roles** en el menú lateral
3. Click "Create role"

**Rol 1: Administrador**
- **Role name**: `admin`
- **Description**: `Administrador con acceso completo`
- Click "Save"

**Rol 2: Editor**
- **Role name**: `editor`
- **Description**: `Usuario que puede modificar contenido`
- Click "Save"

**Rol 3: Lector**
- **Role name**: `lector`
- **Description**: `Usuario con acceso solo de lectura`
- Click "Save"

### Paso 1.2: Crear roles de cliente (opcional)

1. Ir a **Clients** → `app-practica`
2. Ir a la pestaña **Roles**
3. Click "Create role"

**Rol específico del cliente:**
- **Role name**: `api-access`
- **Description**: `Acceso a la API`
- Click "Save"

### Paso 1.3: Asignar roles a usuarios

**Usuario: bruno (Admin)**
1. Ir a **Users** → buscar `bruno`
2. Ir a la pestaña **Role mapping**
3. Click "Assign role"
4. Seleccionar `admin`
5. Click "Assign"

**Usuario: alumno1 (Lector)**
1. Ir a **Users** → buscar `alumno1`
2. Ir a la pestaña **Role mapping**
3. Click "Assign role"
4. Seleccionar `lector`
5. Click "Assign"

**Crear nuevo usuario: editor1 (Editor)**
1. Ir a **Users** → "Add user"
2. Username: `editor1`
3. Email: `editor1@unpsjb.edu.ar`
4. Click "Create"
5. Establecer contraseña: `Password123`
6. Ir a **Role mapping**
7. Asignar rol `editor`

---

## Parte 2: Incluir Roles en Tokens (15 min)

### Paso 2.1: Configurar mappers de cliente

Los **mappers** permiten incluir información adicional en los tokens.

1. Ir a **Clients** → `app-practica`
2. Ir a la pestaña **Client scopes**
3. Click en `app-practica-dedicated`
4. Ir a la pestaña **Mappers**
5. Click "Add mapper" → "By configuration"
6. Seleccionar "User Realm Role"

**Configuración del mapper:**
- **Name**: `realm-roles`
- **Token Claim Name**: `roles`
- **Claim JSON Type**: String
- **Add to ID token**: ON
- **Add to access token**: ON
- **Add to userinfo**: ON
- Click "Save"

### Paso 2.2: Verificar roles en tokens

**Obtener token para bruno (admin):**

```bash
curl -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "username=bruno" \
  -d "password=Password123" | jq
```

Copiar el `access_token` y decodificarlo en https://jwt.io

Buscar el claim `roles`:
```json
{
  "realm_access": {
    "roles": [
      "admin",
      "default-roles-arys-curso",
      "offline_access",
      "uma_authorization"
    ]
  }
}
```

**Obtener token para alumno1 (lector):**

```bash
curl -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "username=alumno1" \
  -d "password=Password123" | jq
```

Decodificar y verificar que solo tiene el rol `lector`.

---

## Parte 3: API Protegida con RBAC (25 min)

### Objetivo

Implementar una API simple que valide roles y permita/deniegue acceso según el rol del usuario.

### Paso 3.1: Crear API con Node.js y Express

**Instalar dependencias:**

```bash
mkdir api-rbac
cd api-rbac
npm init -y
npm install express jsonwebtoken jwks-rsa
```

**Crear archivo `server.js`:**

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

const app = express();
const PORT = 3000;

// Configurar cliente JWKS para validar tokens
const client = jwksClient({
  jwksUri: 'http://localhost:8080/realms/arys-curso/protocol/openid-connect/certs'
});

function getKey(header, callback) {
  client.getSigningKey(header.kid, function(err, key) {
    if (err) {
      callback(err);
    } else {
      const signingKey = key.publicKey || key.rsaPublicKey;
      callback(null, signingKey);
    }
  });
}

// Middleware para validar token
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Token no proporcionado' });
  }

  jwt.verify(token, getKey, {
    audience: 'account',
    issuer: 'http://localhost:8080/realms/arys-curso',
    algorithms: ['RS256']
  }, (err, decoded) => {
    if (err) {
      return res.status(403).json({ error: 'Token inválido' });
    }
    req.user = decoded;
    next();
  });
}

// Middleware para verificar roles
function requireRole(...roles) {
  return (req, res, next) => {
    const userRoles = req.user.realm_access?.roles || [];
    
    const hasRole = roles.some(role => userRoles.includes(role));
    
    if (!hasRole) {
      return res.status(403).json({ 
        error: 'Acceso denegado',
        message: `Se requiere uno de estos roles: ${roles.join(', ')}`,
        yourRoles: userRoles
      });
    }
    
    next();
  };
}

// Rutas públicas
app.get('/', (req, res) => {
  res.json({ message: 'API de Control de Acceso - RBAC' });
});

// Ruta protegida - Solo lectura (todos los roles autenticados)
app.get('/api/users', authenticateToken, (req, res) => {
  res.json({
    message: 'Lista de usuarios',
    users: [
      { id: 1, name: 'Bruno Zappellini' },
      { id: 2, name: 'Alumno Uno' },
      { id: 3, name: 'Editor Uno' }
    ]
  });
});

// Ruta protegida - Solo editor y admin pueden modificar
app.post('/api/users', authenticateToken, requireRole('editor', 'admin'), (req, res) => {
  res.json({
    message: 'Usuario creado exitosamente',
    user: { id: 4, name: 'Nuevo Usuario' }
  });
});

// Ruta protegida - Solo admin puede eliminar
app.delete('/api/users/:id', authenticateToken, requireRole('admin'), (req, res) => {
  res.json({
    message: `Usuario ${req.params.id} eliminado exitosamente`
  });
});

// Ruta para ver información del usuario actual
app.get('/api/me', authenticateToken, (req, res) => {
  res.json({
    username: req.user.preferred_username,
    email: req.user.email,
    roles: req.user.realm_access?.roles || [],
    sub: req.user.sub
  });
});

app.listen(PORT, () => {
  console.log(`API corriendo en http://localhost:${PORT}`);
});
```

**Ejecutar el servidor:**

```bash
node server.js
```

### Paso 3.2: Probar la API

**Obtener tokens para cada usuario:**

```bash
# Token de bruno (admin)
ADMIN_TOKEN=$(curl -s -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "username=bruno" \
  -d "password=Password123" | jq -r '.access_token')

# Token de alumno1 (lector)
READER_TOKEN=$(curl -s -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "username=alumno1" \
  -d "password=Password123" | jq -r '.access_token')

# Token de editor1 (editor)
EDITOR_TOKEN=$(curl -s -X POST http://localhost:8080/realms/arys-curso/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=password" \
  -d "client_id=app-practica" \
  -d "client_secret=CLIENT_SECRET" \
  -d "username=editor1" \
  -d "password=Password123" | jq -r '.access_token')
```

**Probar acceso de lectura (todos deberían poder):**

```bash
# Con admin
curl http://localhost:3000/api/users \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq

# Con lector
curl http://localhost:3000/api/users \
  -H "Authorization: Bearer $READER_TOKEN" | jq

# Con editor
curl http://localhost:3000/api/users \
  -H "Authorization: Bearer $EDITOR_TOKEN" | jq
```

**Probar creación de usuarios (solo editor y admin):**

```bash
# Con admin (✅ Debe funcionar)
curl -X POST http://localhost:3000/api/users \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq

# Con editor (✅ Debe funcionar)
curl -X POST http://localhost:3000/api/users \
  -H "Authorization: Bearer $EDITOR_TOKEN" | jq

# Con lector (❌ Debe fallar)
curl -X POST http://localhost:3000/api/users \
  -H "Authorization: Bearer $READER_TOKEN" | jq
```

**Probar eliminación de usuarios (solo admin):**

```bash
# Con admin (✅ Debe funcionar)
curl -X DELETE http://localhost:3000/api/users/1 \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq

# Con editor (❌ Debe fallar)
curl -X DELETE http://localhost:3000/api/users/1 \
  -H "Authorization: Bearer $EDITOR_TOKEN" | jq

# Con lector (❌ Debe fallar)
curl -X DELETE http://localhost:3000/api/users/1 \
  -H "Authorization: Bearer $READER_TOKEN" | jq
```

**Ver información del usuario actual:**

```bash
curl http://localhost:3000/api/me \
  -H "Authorization: Bearer $ADMIN_TOKEN" | jq
```

---

## Ejercicios Adicionales

### Ejercicio 1: Roles compuestos

**Crear un rol compuesto que incluya otros roles:**

1. En Keycloak, ir a **Realm roles**
2. Crear nuevo rol: `superadmin`
3. Activar "Composite"
4. Agregar `admin`, `editor`, `lector`
5. Asignar `superadmin` a un usuario
6. Verificar que el usuario tiene todos los roles

### Ejercicio 2: Roles específicos de cliente

**Implementar roles específicos para la API:**

1. Ir a **Clients** → `app-practica` → **Roles**
2. Crear roles:
   - `read:users`
   - `write:users`
   - `delete:users`
3. Modificar el código para validar estos roles específicos

### Ejercicio 3: Jerarquía de roles

**Diseñar una jerarquía de permisos:**

```
superadmin
  └─ admin
      └─ editor
          └─ lector
```

Implementar lógica que permita herencia de permisos.

---

## Verificación y Evaluación

### Checklist de Verificación

- [ ] Roles creados en Keycloak (admin, editor, lector)
- [ ] Roles asignados a usuarios correctamente
- [ ] Mappers configurados para incluir roles en tokens
- [ ] Tokens decodificados muestran roles correctos
- [ ] API implementada con validación de tokens
- [ ] Middleware de roles funcionando
- [ ] Lector puede ver pero no modificar
- [ ] Editor puede ver y modificar pero no eliminar
- [ ] Admin puede realizar todas las operaciones
- [ ] Respuestas HTTP correctas (200, 403)

### Tabla de Resultados Esperados

| Usuario | Rol | GET /api/users | POST /api/users | DELETE /api/users/1 |
|---------|-----|----------------|-----------------|---------------------|
| bruno | admin | ✅ 200 | ✅ 200 | ✅ 200 |
| editor1 | editor | ✅ 200 | ✅ 200 | ❌ 403 |
| alumno1 | lector | ✅ 200 | ❌ 403 | ❌ 403 |
| (sin token) | - | ❌ 401 | ❌ 401 | ❌ 401 |

### Preguntas de Evaluación

1. **¿Qué ventajas ofrece RBAC sobre ACL tradicionales?**

2. **¿Cómo se incluyen los roles en los tokens JWT de Keycloak?**

3. **¿Por qué es importante validar la firma del JWT antes de confiar en los roles?**

4. **¿Cuál es la diferencia entre roles de realm y roles de cliente?**

5. **¿Cómo implementarías RBAC en una aplicación de frontend (React/Vue)?**

---

## Conclusiones

Al completar este laboratorio, has:

✅ Definido roles y políticas de acceso en Keycloak  
✅ Asignado roles a usuarios de forma centralizada  
✅ Configurado tokens para incluir información de roles  
✅ Implementado una API con control de acceso basado en roles  
✅ Validado tokens JWT y verificado permisos  
✅ Comprendido las diferencias entre autenticación y autorización

**Próximos pasos:**
- Explorar ABAC (Attribute-Based Access Control)
- Implementar políticas más complejas con contexto
- Integrar RBAC en aplicaciones de frontend
- Configurar SSO entre múltiples aplicaciones
