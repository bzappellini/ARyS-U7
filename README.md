# ARyS-U7 🔐
Servicios de Autenticación y Control de Acceso (AAA)

## 📖 Descripción

Este repositorio contiene todo el material educativo para la Unidad 7 de ARyS (Administración de Redes y Servicios), enfocado en servicios AAA (Authentication, Authorization, Accounting).

## 🌐 Acceso Web

### 🎯 Portal Principal
Accede a todo el contenido desde la web:

**https://bzappellini.github.io/ARyS-U7/**

### 📊 Filminas (Presentaciones con Reveal.js)
- [Introducción a AAA](https://bzappellini.github.io/ARyS-U7/slides.html?md=../filminas/introduccion-aaa.md)
- [Protocolos AAA](https://bzappellini.github.io/ARyS-U7/slides.html?md=../filminas/protocolos-aaa.md)
- [Implementación de AAA](https://bzappellini.github.io/ARyS-U7/slides.html?md=../filminas/implementacion-aaa.md)

### 📚 Material Teórico
- [Wiki Integrada](https://bzappellini.github.io/ARyS-U7/wiki.html) - Contenido teórico completo
- [GitHub Wiki](https://github.com/bzappellini/ARyS-U7/wiki) - Documentación en GitHub

### 💻 Material Práctico
- [Trabajos Prácticos](https://bzappellini.github.io/ARyS-U7/practicos.html) - Laboratorios guiados

## 📂 Estructura del Repositorio

```
ARyS-U7/
├── filminas/           # Presentaciones en formato Markdown para Reveal.js
│   ├── introduccion-aaa.md
│   ├── protocolos-aaa.md
│   └── implementacion-aaa.md
├── teoria/             # Material teórico en Markdown
│   └── 01-introduccion-aaa.md
├── practicos/          # Guías de laboratorio
│   └── laboratorio-01.md
├── wiki/               # Contenido para GitHub Wiki
│   ├── Home.md
│   └── Introduccion-AAA.md
├── docs/               # Sitio web (GitHub Pages)
│   ├── index.html      # Portal principal
│   ├── slides.html     # Visor de presentaciones
│   ├── wiki.html       # Wiki integrada
│   └── practicos.html  # Listado de trabajos prácticos
└── README.md           # Este archivo
```

## 🚀 Características

### ✨ Presentaciones Interactivas
- Powered by [Reveal.js](https://revealjs.com/)
- Navegación con teclado (flechas)
- Modo presentador (presiona 'S')
- Vista general (presiona 'O')
- Notas para el presentador
- Código con syntax highlighting

### 📖 Wiki Completa
- Contenido teórico organizado
- Navegación lateral
- Búsqueda de contenido
- Enlaces cruzados
- Formato profesional

### 🧪 Laboratorios Prácticos
- 6 laboratorios completos
- Dificultad progresiva (básico → avanzado)
- Instrucciones paso a paso
- Ejercicios y evaluación

## 🎯 Temario

### Módulo 1: Fundamentos AAA
- ¿Qué es AAA?
- Autenticación, Autorización, Auditoría
- Arquitectura y componentes
- Casos de uso

### Módulo 2: Protocolos AAA
- RADIUS: Características y uso
- TACACS+: Control de dispositivos
- Diameter: Redes modernas
- Comparación de protocolos

### Módulo 3: Implementación
- Instalación de FreeRADIUS
- Configuración básica
- Integración con LDAP/AD
- WiFi 802.1X
- VPN con AAA

### Módulo 4: Casos Prácticos
- WiFi empresarial
- Administración de dispositivos
- Network Access Control (NAC)
- Alta disponibilidad

## 🛠️ Tecnologías Utilizadas

- **Reveal.js**: Presentaciones HTML
- **Markdown**: Contenido y documentación
- **GitHub Pages**: Hosting del sitio
- **HTML/CSS/JavaScript**: Portal web
- **FreeRADIUS**: Laboratorios prácticos

## 📝 Cómo Usar Este Material

### Para Estudiantes

1. **Revisar las presentaciones**: Comienza con las filminas para una visión general
2. **Leer la wiki**: Profundiza en los conceptos teóricos
3. **Realizar los laboratorios**: Practica con los trabajos prácticos
4. **Consultar referencias**: Usa los enlaces a RFCs y documentación oficial

### Para Docentes

1. **Presentar con Reveal.js**: Usa las filminas en clase
2. **Asignar laboratorios**: Los prácticos están listos para usar
3. **Extender contenido**: Todo es Markdown, fácil de modificar
4. **Clonar y personalizar**: Fork el repo y adapta a tu curso

## 🔧 Desarrollo Local

### Opción 1: Servidor HTTP Simple
```bash
# Con Python 3
cd docs
python3 -m http.server 8000

# Acceder en http://localhost:8000
```

### Opción 2: Live Server (VS Code)
1. Instala la extensión "Live Server"
2. Abre la carpeta `docs`
3. Click derecho en `index.html` → "Open with Live Server"

### Opción 3: Nginx/Apache
```bash
# Configurar document root a /path/to/ARyS-U7/docs
```

## 📚 Referencias y Recursos

### Estándares
- [RFC 2865 - RADIUS](https://tools.ietf.org/html/rfc2865)
- [RFC 2866 - RADIUS Accounting](https://tools.ietf.org/html/rfc2866)
- [RFC 6733 - Diameter](https://tools.ietf.org/html/rfc6733)
- [IEEE 802.1X](https://standards.ieee.org/standard/802_1X-2020.html)

### Software
- [FreeRADIUS](https://freeradius.org/) - Servidor RADIUS open source
- [Cisco ISE](https://www.cisco.com/c/en/us/products/security/identity-services-engine/) - Solución comercial
- [PacketFence](https://www.packetfence.org/) - NAC open source

### Aprendizaje
- [FreeRADIUS Documentation](https://freeradius.org/documentation/)
- [NetworkRADIUS](https://networkradius.com/) - Recursos y soporte
- [RADIUS RFC Wiki](https://freeradius.org/rfc/)

## 🤝 Contribuciones

Este material es educativo y está en constante mejora. Si encuentras errores o tienes sugerencias:

1. Abre un [Issue](https://github.com/bzappellini/ARyS-U7/issues)
2. Envía un Pull Request
3. Contacta al docente

## 📄 Licencia

Este material es para uso educativo en la materia ARyS (Administración de Redes y Servicios).

## 👨‍🏫 Autor

Cátedra de Administración de Redes y Servicios (ARyS)

---

**⭐ Si este material te resulta útil, dale una estrella al repositorio!**
