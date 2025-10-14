# Servicios AAA
## Autenticación, Autorización y Auditoría

---

## ¿Qué es AAA?

AAA es un framework de seguridad que controla:

- **Autenticación** (Authentication): ¿Quién eres?
- **Autorización** (Authorization): ¿Qué puedes hacer?
- **Auditoría** (Accounting): ¿Qué hiciste?

---

## Autenticación

### Verificación de identidad

- Usuarios
- Dispositivos
- Servicios

### Métodos comunes:
- Contraseñas
- Certificados digitales
- Tokens
- Biometría

---

## Autorización

### Control de acceso

Una vez autenticado, ¿qué permisos tiene el usuario?

- Acceso a recursos
- Nivel de privilegios
- Restricciones temporales
- Políticas de seguridad

---

## Auditoría (Accounting)

### Registro de actividades

- ¿Qué recursos se utilizaron?
- ¿Cuándo se accedió?
- ¿Qué acciones se realizaron?
- ¿Cuánto tiempo duró la sesión?

### Propósitos:
- Facturación
- Análisis de seguridad
- Cumplimiento normativo
- Resolución de problemas

---

## Arquitectura AAA

```
┌─────────┐         ┌──────────────┐         ┌──────────┐
│ Cliente │ ────▶   │ Servidor NAS │ ────▶   │ Servidor │
│         │         │   (Network   │         │   AAA    │
└─────────┘         │    Access    │         └──────────┘
                    │   Server)    │
                    └──────────────┘
```

---

## Beneficios de AAA

- 🔒 **Seguridad centralizada**
- 📊 **Control y visibilidad**
- ⚖️ **Cumplimiento normativo**
- 🔄 **Gestión simplificada**
- 📈 **Escalabilidad**

---

## Casos de uso

1. **Acceso a la red**: Control de acceso WiFi
2. **VPN**: Autenticación de usuarios remotos
3. **Switches y routers**: Control de administración
4. **Aplicaciones**: Single Sign-On (SSO)
5. **Servicios en la nube**: Gestión de identidades

---

## Próximos pasos

- Protocolos AAA (RADIUS, TACACS+, Diameter)
- Implementación práctica
- Configuración de servidores AAA
- Mejores prácticas de seguridad

---

# ¿Preguntas?

📧 Contacto
🔗 [Volver al índice](../docs/index.html)
