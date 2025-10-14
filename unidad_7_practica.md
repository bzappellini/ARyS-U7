# 🧪 Unidad 7 — Actividades Prácticas (3 horas)

**Carrera:** Licenciatura en Sistemas  
**Materia:** Administración de Redes y Seguridad  
**Sede:** Trelew — UNPSJB  
**Duración:** 3 h prácticas  

---

## 1️⃣ Laboratorio: LDAP y autenticación básica (45 min)

**Objetivo:** Comprender el funcionamiento de un servicio de directorio y la autenticación básica de usuarios.

**Actividades:**
1. Instalar y configurar un servidor **OpenLDAP**.  
2. Crear usuarios y grupos dentro del árbol LDAP.  
3. Probar autenticación y búsqueda de usuarios con los siguientes comandos:

```bash
ldapsearch -x -b "dc=ejemplo,dc=com"
ldapwhoami -x -D "cn=usuario,dc=ejemplo,dc=com" -W
```

**Puntos de análisis:**
- Estructura del DN (Distinguished Name).  
- Verificación de usuarios autenticados.  
- Visualizar resultados del árbol y atributos.

---

## 2️⃣ Laboratorio: Kerberos (30 min)

**Objetivo:** Comprender la autenticación basada en tickets y la gestión de credenciales.

**Actividades:**
1. Conectar un cliente Linux a un servidor **KDC (Key Distribution Center)**.  
2. Autenticar un usuario y verificar tickets.

**Comandos útiles:**
```bash
kinit usuario
klist
kdestroy
```

**Análisis sugerido:**
- Observar el contenido del ticket.  
- Analizar los tiempos de expiración.  
- Probar autenticaciones repetidas y verificar renovación del ticket.

---

## 3️⃣ Laboratorio: OAuth 2.0 / OIDC con Keycloak (1 hora)

**Objetivo:** Implementar un flujo de autorización moderno usando OAuth 2.0 / OpenID Connect.

**Actividades:**
1. Crear un **realm** y un **client** en Keycloak.  
2. Configurar un flujo de autorización **Authorization Code Flow**.  
3. Obtener tokens usando **Postman** o **curl**:

```bash
curl -X POST \
-d "client_id=app" \
-d "client_secret=clave" \
-d "grant_type=authorization_code" \
-d "code=XYZ" \
-d "redirect_uri=http://localhost/callback" \
https://keycloak.local/realms/curso/protocol/openid-connect/token
```

**Análisis sugerido:**
- Examinar el contenido del **Access Token** e **ID Token** (usar https://jwt.io).  
- Identificar los *claims* de cada token.  
- Diferenciar los flujos de autorización y autenticación.

---

## 4️⃣ Laboratorio: Control de acceso con roles (45 min)

**Objetivo:** Aplicar modelos de control de acceso (RBAC) en un entorno real.

**Actividades:**
1. Definir **roles** y **permisos** en Keycloak.  
2. Asignar usuarios a roles definidos.  
3. Probar acceso diferenciado a una API o aplicación web.

**Ejemplo:**
- Usuario con rol **admin** puede listar y modificar usuarios.  
- Usuario con rol **lectura** solo puede visualizar.

**Verificación:**
- Usar una API protegida por Keycloak.  
- Validar los *Access Token* según los roles asignados.  
- Confirmar las respuestas HTTP (200 OK / 403 Forbidden).

---

✅ **Resultado esperado:**
Al finalizar, los estudiantes deben comprender cómo funcionan los distintos mecanismos de autenticación (LDAP, Kerberos, OAuth/OIDC) y cómo aplicar políticas de control de acceso (RBAC) en un entorno centralizado de identidad.

