# Laboratorio 1: Instalación y Configuración Básica de FreeRADIUS

## Información General

- **Dificultad**: Básica
- **Duración estimada**: 60 minutos
- **Prerrequisitos**: Conocimientos básicos de Linux

## Objetivos

Al completar este laboratorio, serás capaz de:

1. Instalar FreeRADIUS en un sistema Linux
2. Configurar usuarios locales en FreeRADIUS
3. Configurar un cliente RADIUS
4. Probar la autenticación usando radtest
5. Interpretar logs de autenticación

## Requisitos

### Hardware/Software
- Máquina virtual o física con:
  - Ubuntu 20.04/22.04 LTS (recomendado) o Debian
  - 1 GB RAM mínimo
  - 10 GB espacio en disco
  - Acceso a Internet para descarga de paquetes

### Permisos
- Acceso root o sudo

## Parte 1: Instalación

### Paso 1.1: Actualizar el sistema

```bash
sudo apt update
sudo apt upgrade -y
```

### Paso 1.2: Instalar FreeRADIUS

```bash
sudo apt install freeradius freeradius-utils -y
```

### Paso 1.3: Verificar la instalación

```bash
# Verificar versión instalada
freeradius -v

# Verificar estado del servicio
sudo systemctl status freeradius
```

**Resultado esperado**: FreeRADIUS debe estar instalado y el servicio activo.

## Parte 2: Configuración de Usuarios

### Paso 2.1: Detener el servicio

```bash
sudo systemctl stop freeradius
```

### Paso 2.2: Editar el archivo de usuarios

```bash
sudo nano /etc/freeradius/3.0/users
```

### Paso 2.3: Agregar usuarios de prueba

Agrega las siguientes líneas al inicio del archivo (después de los comentarios):

```
# Usuario básico
testuser Cleartext-Password := "testpass"

# Usuario con atributos adicionales
admin Cleartext-Password := "admin123"
    Service-Type = Administrative-User,
    Reply-Message = "Bienvenido Administrador"

# Usuario con restricción de tiempo
temporal Cleartext-Password := "temp123"
    Session-Timeout = 3600,
    Reply-Message = "Sesion valida por 1 hora"
```

Guarda y cierra el archivo (Ctrl+X, Y, Enter).

## Parte 3: Configuración de Clientes RADIUS

### Paso 3.1: Editar archivo de clientes

```bash
sudo nano /etc/freeradius/3.0/clients.conf
```

### Paso 3.2: Configurar cliente localhost

Busca la sección de localhost y verifica/modifica:

```
client localhost {
    ipaddr = 127.0.0.1
    secret = testing123
    require_message_authenticator = no
    nas_type = other
}
```

### Paso 3.3: Agregar cliente de prueba (opcional)

Si deseas probar desde otra máquina:

```
client maquina-prueba {
    ipaddr = 192.168.1.50
    secret = mi_secreto_seguro
    shortname = test-client
}
```

Guarda y cierra el archivo.

## Parte 4: Verificación de Configuración

### Paso 4.1: Verificar sintaxis

```bash
sudo freeradius -C
```

**Resultado esperado**: No debe haber errores. Si hay errores, revisa las configuraciones.

## Parte 5: Pruebas de Autenticación

### Paso 5.1: Iniciar FreeRADIUS en modo debug

```bash
sudo freeradius -X
```

**Nota**: Este modo muestra todos los detalles del procesamiento. Es muy útil para aprendizaje y troubleshooting.

### Paso 5.2: Probar autenticación (en otra terminal)

Abre una nueva terminal y ejecuta:

```bash
# Prueba exitosa
radtest testuser testpass localhost 0 testing123

# Prueba con credenciales incorrectas
radtest testuser wrongpass localhost 0 testing123

# Prueba con usuario admin
radtest admin admin123 localhost 0 testing123
```

### Paso 5.3: Analizar resultados

**Autenticación exitosa debe mostrar**:
```
Sent Access-Request Id 123 from 0.0.0.0:12345 to 127.0.0.1:1812
    User-Name = "testuser"
    User-Password = "testpass"
    ...
Received Access-Accept Id 123 from 127.0.0.1:1812
```

**Autenticación fallida debe mostrar**:
```
Received Access-Reject Id 124 from 127.0.0.1:1812
```

### Paso 5.4: Observar logs en modo debug

En la terminal donde está corriendo `freeradius -X`, observa:

1. Recepción del paquete Access-Request
2. Búsqueda del usuario en el archivo users
3. Verificación de la contraseña
4. Decisión: Accept o Reject
5. Envío de respuesta

## Parte 6: Operación en Producción

### Paso 6.1: Detener modo debug

En la terminal con `freeradius -X`, presiona Ctrl+C.

### Paso 6.2: Iniciar servicio normal

```bash
sudo systemctl start freeradius
sudo systemctl enable freeradius
```

### Paso 6.3: Verificar logs normales

```bash
# Ver logs en tiempo real
sudo tail -f /var/log/freeradius/radius.log

# En otra terminal, hacer prueba
radtest testuser testpass localhost 0 testing123
```

## Parte 7: Exploración Adicional

### Paso 7.1: Explorar estructura de directorios

```bash
ls -la /etc/freeradius/3.0/
```

Directorios importantes:
- `mods-available/`: Módulos disponibles
- `mods-enabled/`: Módulos activos (symlinks)
- `sites-available/`: Sitios virtuales disponibles
- `sites-enabled/`: Sitios activos
- `certs/`: Certificados SSL/TLS

### Paso 7.2: Ver configuración principal

```bash
less /etc/freeradius/3.0/radiusd.conf
```

### Paso 7.3: Explorar módulos

```bash
ls /etc/freeradius/3.0/mods-enabled/
```

## Ejercicios Propuestos

### Ejercicio 1: Usuarios Adicionales
Crea tres usuarios más con diferentes atributos:
- Usuario "consultor" con timeout de 2 horas
- Usuario "invitado" con mensaje personalizado
- Usuario "soporte" con tipo de servicio específico

### Ejercicio 2: Cliente Adicional
Si tienes otra máquina o VM en la red:
- Agrega esa máquina como cliente RADIUS
- Prueba autenticación desde esa máquina
- Analiza los logs

### Ejercicio 3: Troubleshooting
Introduce un error intencional en la configuración:
- Usuario sin contraseña
- Cliente con IP incorrecta
- Secreto que no coincide

Practica identificar y resolver el problema usando modo debug.

## Preguntas de Comprensión

1. ¿Cuál es la diferencia entre `radtest` y una autenticación real desde un AP o switch?
2. ¿Por qué es importante el "secret" compartido?
3. ¿Qué información viaja en un paquete Access-Request?
4. ¿Qué significa el atributo Service-Type?
5. ¿Cuál es la ventaja del modo debug (-X)?

## Entregables

### Documento de Laboratorio
Debe incluir:

1. **Capturas de pantalla**:
   - Instalación completada
   - Archivo users con configuraciones
   - FreeRADIUS en modo debug
   - Pruebas de radtest (exitosa y fallida)
   - Logs mostrando el flujo completo

2. **Análisis**:
   - Explicación del flujo de autenticación observado
   - Identificación de componentes del protocolo RADIUS
   - Respuestas a preguntas de comprensión

3. **Configuraciones**:
   - Archivo users completo
   - Archivo clients.conf (sin secretos reales si es en red pública)

## Troubleshooting Común

### Problema 1: Servicio no inicia
```bash
# Ver errores específicos
sudo freeradius -X
# Verificar permisos
sudo chown -R freerad:freerad /var/log/freeradius/
```

### Problema 2: radtest no funciona
```bash
# Verificar que freeradius esté escuchando
sudo netstat -lunp | grep 1812
# Verificar firewall
sudo ufw status
```

### Problema 3: Autenticación falla sin razón aparente
```bash
# Modo debug es tu mejor amigo
sudo freeradius -X
# Verifica que el usuario esté exactamente como en el archivo
# Verifica el secreto del cliente
```

## Recursos Adicionales

- [FreeRADIUS Wiki](https://wiki.freeradius.org/)
- [Man page de radtest](https://freeradius.org/radiusd/man/radtest.html)
- [Debugging con FreeRADIUS](https://wiki.freeradius.org/guide/HOWTO#debugging)

## Conclusión

Has completado exitosamente la instalación y configuración básica de FreeRADIUS. Este es el fundamento sobre el cual construirás configuraciones más avanzadas en los siguientes laboratorios.

**Próximo paso**: Laboratorio 2 - Integración con LDAP/Active Directory
