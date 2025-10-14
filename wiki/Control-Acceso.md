# Modelos de Control de Acceso

El control de acceso determina qué usuarios pueden acceder a qué recursos y qué operaciones pueden realizar.

---

## ACL (Access Control Lists)

### Definición

Las **Listas de Control de Acceso** definen permisos específicos para cada usuario sobre cada recurso.

### Características

- **Granularidad**: Control por recurso individual
- **Simplicidad**: Fácil de entender
- **Flexibilidad**: Permisos arbitrarios por objeto

### Estructura Básica

```
Recurso: archivo.txt
  - usuario1: lectura, escritura
  - usuario2: solo lectura
  - grupo_admin: lectura, escritura, ejecución, eliminación
```

### Ejemplo en Sistema de Archivos (Linux)

```bash
# Ver permisos
ls -l archivo.txt
-rw-r--r-- 1 bruno profesores 1234 Oct 14 10:30 archivo.txt

# Cambiar permisos
chmod 644 archivo.txt    # rw-r--r--
chmod 755 script.sh      # rwxr-xr-x

# ACLs extendidas
setfacl -m u:alumno1:rw archivo.txt     # Dar rw a alumno1
setfacl -m g:estudiantes:r archivo.txt  # Dar r a grupo
getfacl archivo.txt                      # Ver ACL
```

### Ejemplo en Redes (Cisco ACL)

```
! ACL extendida
access-list 101 permit tcp 192.168.1.0 0.0.0.255 any eq 443
access-list 101 permit tcp 192.168.1.0 0.0.0.255 any eq 80
access-list 101 deny ip any any

! Aplicar a interfaz
interface GigabitEthernet0/0
 ip access-group 101 in
```

### Ventajas

✅ Control directo sobre permisos  
✅ Fácil de implementar en sistemas pequeños  
✅ Permisos específicos por recurso  
✅ Sin intermediarios (roles, grupos, etc.)

### Desventajas

❌ No escala bien en organizaciones grandes  
❌ Difícil de auditar ("¿quién tiene acceso a X?")  
❌ Administración descentralizada  
❌ Propenso a errores humanos  
❌ Difícil mantener consistencia

---

## RBAC (Role-Based Access Control)

### Definición

El **Control de Acceso Basado en Roles** asigna permisos a roles, y usuarios se asignan a roles.

### Modelo Conceptual

```
Usuarios → asignados a → Roles → tienen → Permisos → sobre → Recursos
```

### Componentes

**1. Roles**
- Conjunto de permisos
- Representa función o responsabilidad
- Ejemplos: Admin, Editor, Lector, Moderador

**2. Permisos**
- Operaciones sobre recursos
- Ejemplos: crear, leer, actualizar, eliminar

**3. Usuarios**
- Individuos que acceden al sistema
- Pueden tener múltiples roles

**4. Recursos**
- Objetos protegidos
- Ejemplos: archivos, APIs, bases de datos

### Ejemplo Práctico

**Roles definidos:**
```
Admin:
  - crear usuarios
  - eliminar usuarios
  - modificar usuarios
  - leer usuarios
  - modificar configuración

Editor:
  - crear contenido
  - modificar contenido
  - leer contenido
  - publicar contenido

Lector:
  - leer contenido público
```

**Asignación de usuarios:**
```
bruno@unpsjb.edu.ar → Admin
editor1@unpsjb.edu.ar → Editor
alumno1@unpsjb.edu.ar → Lector
```

### RBAC Jerárquico

Los roles pueden heredar permisos de otros roles.

```
SuperAdmin
  └─ Admin
      └─ Editor
          └─ Lector
```

**Herencia:**
- SuperAdmin tiene todos los permisos de Admin, Editor, Lector
- Admin tiene permisos de Editor y Lector
- Editor tiene permisos de Lector

### Implementación en Base de Datos

```sql
-- Tabla de roles
CREATE TABLE roles (
  id INT PRIMARY KEY,
  name VARCHAR(50),
  description TEXT
);

-- Tabla de permisos
CREATE TABLE permissions (
  id INT PRIMARY KEY,
  resource VARCHAR(50),
  action VARCHAR(50)
);

-- Relación roles-permisos (muchos a muchos)
CREATE TABLE role_permissions (
  role_id INT,
  permission_id INT,
  FOREIGN KEY (role_id) REFERENCES roles(id),
  FOREIGN KEY (permission_id) REFERENCES permissions(id)
);

-- Relación usuarios-roles (muchos a muchos)
CREATE TABLE user_roles (
  user_id INT,
  role_id INT,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (role_id) REFERENCES roles(id)
);
```

### Implementación en Keycloak

**Crear roles:**
```
Realm roles → Create role
- admin
- editor
- lector
```

**Asignar a usuarios:**
```
Users → [usuario] → Role mapping → Assign role
```

**En tokens JWT:**
```json
{
  "realm_access": {
    "roles": ["admin", "editor"]
  }
}
```

### Validación en Aplicación

**Backend (Node.js/Express):**
```javascript
function requireRole(...roles) {
  return (req, res, next) => {
    const userRoles = req.user.roles || [];
    const hasRole = roles.some(r => userRoles.includes(r));
    
    if (!hasRole) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
}

// Uso
app.delete('/api/users/:id', 
  authenticateToken,
  requireRole('admin'),
  (req, res) => {
    // Solo admin puede eliminar
  }
);
```

**Frontend (React):**
```javascript
function DeleteButton({ user, currentUserRoles }) {
  const canDelete = currentUserRoles.includes('admin');
  
  if (!canDelete) return null;
  
  return <button onClick={handleDelete}>Eliminar</button>;
}
```

### Ventajas

✅ **Escalabilidad**: Fácil gestionar miles de usuarios  
✅ **Mantenimiento**: Cambiar permisos de rol afecta a todos  
✅ **Auditoría**: "¿Qué puede hacer el rol X?"  
✅ **Onboarding/Offboarding**: Asignar/revocar roles rápidamente  
✅ **Separación de responsabilidades**: Roles bien definidos  
✅ **Cumplimiento**: Facilita certificaciones (SOC2, ISO 27001)

### Desventajas

❌ No considera contexto (hora, ubicación, etc.)  
❌ Roles pueden proliferar ("role explosion")  
❌ Difícil modelar relaciones complejas  
❌ No dinámico (requiere cambios manuales)

---

## ABAC (Attribute-Based Access Control)

### Definición

El **Control de Acceso Basado en Atributos** toma decisiones evaluando atributos del usuario, recurso y contexto.

### Componentes

**1. Atributos de Usuario (Subject)**
- `user.department = "IT"`
- `user.clearance_level = "confidential"`
- `user.employment_status = "active"`

**2. Atributos de Recurso (Object)**
- `document.classification = "confidential"`
- `document.owner = "bruno"`
- `document.created_date = "2024-01-15"`

**3. Atributos de Entorno (Environment)**
- `time = "09:00-18:00"`
- `location = "office_network"`
- `device.managed = true`

**4. Atributos de Acción**
- `action = "read"`
- `action = "modify"`
- `action = "delete"`

### Políticas ABAC

Las políticas son reglas que evalúan atributos.

**Ejemplo 1: Acceso a documentos confidenciales**
```
PERMIT IF:
  user.clearance_level >= document.classification AND
  user.department == document.department AND
  time BETWEEN 08:00 AND 18:00 AND
  location == "office_network"
```

**Ejemplo 2: Aprobación de gastos**
```
PERMIT IF:
  action == "approve" AND
  expense.amount < user.approval_limit AND
  user.role == "manager" AND
  expense.department == user.department
```

**Ejemplo 3: Acceso fuera de horario**
```
PERMIT IF:
  action == "read" AND
  (
    time BETWEEN 08:00 AND 18:00 OR
    user.role == "oncall_engineer"
  )
```

### Lenguajes de Políticas

#### XACML (eXtensible Access Control Markup Language)

```xml
<Policy PolicyId="DocumentAccessPolicy">
  <Target>
    <Resources>
      <Resource>
        <ResourceMatch MatchId="string-equal">
          <AttributeValue>document</AttributeValue>
          <ResourceAttributeDesignator 
            AttributeId="resource-type"/>
        </ResourceMatch>
      </Resource>
    </Resources>
  </Target>
  
  <Rule RuleId="ConfidentialAccess" Effect="Permit">
    <Condition>
      <Apply FunctionId="and">
        <Apply FunctionId="string-equal">
          <AttributeValue>confidential</AttributeValue>
          <SubjectAttributeDesignator 
            AttributeId="clearance-level"/>
        </Apply>
        <Apply FunctionId="time-in-range">
          <CurrentTime/>
          <AttributeValue>08:00:00</AttributeValue>
          <AttributeValue>18:00:00</AttributeValue>
        </Apply>
      </Apply>
    </Condition>
  </Rule>
</Policy>
```

#### OPA Rego (Open Policy Agent)

```rego
package authz

# Permitir si el usuario es el dueño del recurso
allow {
  input.user.id == input.resource.owner_id
}

# Permitir si el usuario es admin
allow {
  input.user.roles[_] == "admin"
}

# Permitir lectura si está en el mismo departamento
allow {
  input.action == "read"
  input.user.department == input.resource.department
}

# Permitir solo durante horario laboral
allow {
  time.now_ns() >= time.parse_ns("15:04", "08:00")
  time.now_ns() <= time.parse_ns("15:04", "18:00")
}
```

#### Cedar (AWS)

```cedar
permit(
  principal in Group::"Engineers",
  action == Action::"read",
  resource in Folder::"projects"
) when {
  context.time >= time("08:00:00") &&
  context.time <= time("18:00:00") &&
  context.ip in ip("10.0.0.0/8")
};
```

### Implementación con OPA

**Instalar OPA:**
```bash
# Docker
docker run -p 8181:8181 openpolicyagent/opa:latest \
  run --server --log-level debug

# Binario
curl -L -o opa https://openpolicyagent.org/downloads/latest/opa_linux_amd64
chmod +x opa
./opa run --server
```

**Política de ejemplo:**
```rego
package api.authz

# Valores por defecto
default allow = false

# Permitir a admins todo
allow {
  input.user.roles[_] == "admin"
}

# Permitir lectura a todos los autenticados
allow {
  input.action == "read"
  input.user.authenticated == true
}

# Permitir modificación solo al dueño
allow {
  input.action == "write"
  input.resource.owner == input.user.id
}

# Permitir solo desde red corporativa
allow {
  net.cidr_contains("10.0.0.0/8", input.context.ip_address)
}
```

**Consultar política:**
```bash
curl -X POST http://localhost:8181/v1/data/api/authz/allow \
  -H "Content-Type: application/json" \
  -d '{
    "input": {
      "user": {
        "id": "bruno",
        "roles": ["editor"],
        "authenticated": true
      },
      "action": "read",
      "resource": {
        "type": "document",
        "owner": "bruno"
      },
      "context": {
        "ip_address": "10.0.1.50"
      }
    }
  }'

# Respuesta
{
  "result": true
}
```

### Ventajas de ABAC

✅ **Flexibilidad extrema**: Políticas muy granulares  
✅ **Dinámico**: Cambios automáticos según contexto  
✅ **Escalable**: No proliferación de roles  
✅ **Expresivo**: Puede modelar cualquier política  
✅ **Centralizado**: Motor de políticas único  
✅ **Auditable**: Justificación de decisiones

### Desventajas de ABAC

❌ **Complejidad**: Difícil de diseñar correctamente  
❌ **Performance**: Evaluación puede ser costosa  
❌ **Debugging**: Difícil entender por qué se denegó  
❌ **Curva de aprendizaje**: Requiere expertise  
❌ **Testing**: Muchos casos de prueba

---

## Comparación de Modelos

| Aspecto | ACL | RBAC | ABAC |
|---------|-----|------|------|
| **Complejidad** | Baja | Media | Alta |
| **Escalabilidad** | Baja | Alta | Muy alta |
| **Flexibilidad** | Baja | Media | Muy alta |
| **Mantenimiento** | Difícil | Medio | Complejo |
| **Auditoría** | Difícil | Fácil | Media |
| **Contexto** | No | No | Sí |
| **Dinamismo** | No | No | Sí |
| **Casos de uso** | Archivos simples | Aplicaciones empresariales | Sistemas complejos |

---

## Modelo Híbrido (Recomendado)

En la práctica, combinar modelos ofrece lo mejor de cada uno.

### RBAC + ABAC

**Estrategia:**
1. **RBAC base**: Roles para permisos generales
2. **ABAC para excepciones**: Contexto y casos especiales

**Ejemplo:**
```rego
package authz

# Base: RBAC
allow {
  input.user.roles[_] == "admin"
}

allow {
  input.user.roles[_] == "editor"
  input.action == "write"
}

# ABAC: Restricciones contextuales
deny {
  # Denegar fuera de horario (excepto admins)
  not input.user.roles[_] == "admin"
  time.now_ns() < time.parse_ns("15:04", "08:00")
}

deny {
  # Denegar acceso desde IPs externas
  not net.cidr_contains("10.0.0.0/8", input.context.ip)
  input.resource.classification == "confidential"
}
```

### ACL para Recursos Específicos + RBAC General

**Ejemplo en CMS:**
- **RBAC**: Roles generales (admin, editor, autor)
- **ACL**: Permisos específicos por artículo

```javascript
function canEditArticle(user, article) {
  // RBAC: Admins pueden todo
  if (user.roles.includes('admin')) return true;
  
  // ACL: Autor puede editar sus propios artículos
  if (article.author_id === user.id) return true;
  
  // ACL: Usuarios explícitos con permiso
  if (article.acl.editors.includes(user.id)) return true;
  
  return false;
}
```

---

## Mejores Prácticas

### Diseño de Políticas

✅ **Principio de mínimo privilegio**: Denegar por defecto  
✅ **Separación de responsabilidades**: Roles bien definidos  
✅ **Revisión periódica**: Auditar permisos regularmente  
✅ **Documentación**: Justificar cada política  
✅ **Testing exhaustivo**: Probar casos positivos y negativos

### Implementación

✅ **Centralizar decisiones**: Motor de autorización único  
✅ **Caching**: Cachear decisiones cuando sea posible  
✅ **Logging**: Registrar todas las decisiones  
✅ **Fail secure**: Ante error, denegar acceso  
✅ **Performance monitoring**: Medir tiempo de evaluación

### Auditoría

✅ **Logs estructurados**: JSON con contexto completo  
✅ **Alertas**: Notificar accesos anómalos  
✅ **Retención**: Mantener logs según regulaciones  
✅ **Análisis**: Revisar patrones de acceso  
✅ **Reportes**: Facilitar auditorías externas

---

## Herramientas y Tecnologías

### Motores de Políticas

- **Open Policy Agent (OPA)**: CNCF, usado por Netflix, Pinterest
- **Cedar**: AWS, usado en Amazon Verified Permissions
- **Casbin**: Go, múltiples modelos (ACL, RBAC, ABAC)
- **Ory Keto**: Relaciones de Google Zanzibar

### Frameworks de Aplicación

- **Spring Security** (Java): RBAC integrado
- **Django Guardian** (Python): ACL por objeto
- **Pundit** (Ruby): Políticas como objetos
- **CASL** (JavaScript): Permissions en frontend

### Soluciones Enterprise

- **Okta Advanced Server Access**: ABAC para infraestructura
- **AWS IAM**: Policies basadas en JSON
- **Azure RBAC**: Roles para recursos Azure
- **Google IAM**: Roles y condiciones

---

## Casos de Estudio

### Caso 1: Sistema de Salud

**Requisitos:**
- Médicos ven pacientes asignados
- Enfermeras ven pacientes de su piso
- Admin ve todo
- Auditoría completa

**Solución: RBAC + ABAC**
```rego
allow {
  input.user.role == "doctor"
  input.patient.id in input.user.assigned_patients
}

allow {
  input.user.role == "nurse"
  input.patient.floor == input.user.floor
}

allow {
  input.user.role == "admin"
}
```

### Caso 2: Plataforma de Colaboración

**Requisitos:**
- Dueño de documento controla acceso
- Compartir con usuarios/grupos
- Permisos heredables en carpetas

**Solución: ACL + RBAC**
- RBAC para roles admin/user
- ACL para compartir específico

### Caso 3: Infraestructura Cloud

**Requisitos:**
- Acceso según proyecto
- Restricciones de red y horario
- MFA obligatorio para producción

**Solución: ABAC completo**
```rego
allow {
  input.resource.environment == "production"
  input.user.mfa_verified == true
  input.user.project == input.resource.project
  time.now_ns() < input.resource.maintenance_window
}
```

---

## Recursos Adicionales

### Estándares
- NIST RBAC Model
- XACML Specification
- OASIS Standards

### Documentación
- [OPA Documentation](https://www.openpolicyagent.org/docs/)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [NIST SP 800-162 - ABAC](https://csrc.nist.gov/publications/detail/sp/800-162/final)

### Cursos y Tutoriales
- OPA Styra Academy
- AWS Identity and Access Management
- Microsoft Learn - Azure RBAC
