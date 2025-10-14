# 🚀 Guía de Configuración

Este documento explica cómo configurar y desplegar el sitio web de ARyS-U7.

## 📋 Configuración de GitHub Pages

### Paso 1: Habilitar GitHub Pages

1. Ve a la configuración del repositorio: `Settings` → `Pages`
2. En "Source", selecciona: **GitHub Actions**
3. Guarda los cambios

### Paso 2: Verificar el Despliegue

1. Ve a la pestaña `Actions` en el repositorio
2. Verifica que el workflow "Deploy to GitHub Pages" se ejecute correctamente
3. Una vez completado, el sitio estará disponible en:
   ```
   https://bzappellini.github.io/ARyS-U7/
   ```

## 🔧 Configuración de GitHub Wiki

### Paso 1: Inicializar la Wiki

1. Ve a la pestaña `Wiki` en el repositorio
2. Haz clic en "Create the first page"
3. Crea la página inicial (Home)

### Paso 2: Copiar Contenido

Copia el contenido de los archivos en la carpeta `wiki/` a las páginas correspondientes en GitHub Wiki:

```bash
wiki/Home.md → Home (página principal de la wiki)
wiki/Introduccion-AAA.md → Nueva página: "Introduccion-AAA"
```

### Paso 3: Agregar Páginas Adicionales

Para cada tema del curso, crea una nueva página en la Wiki con el contenido teórico correspondiente.

## 📁 Estructura de Archivos

### Filminas (Presentaciones)
Los archivos Markdown en `filminas/` se convierten automáticamente en presentaciones con Reveal.js:

```
filminas/introduccion-aaa.md → Accesible vía web
```

Para agregar nuevas presentaciones:
1. Crea un archivo `.md` en `filminas/`
2. Usa `---` para separar slides
3. Agrega el enlace en `docs/index.html`

### Material Teórico
Los archivos en `teoria/` son material de referencia en Markdown:

```
teoria/01-introduccion-aaa.md
```

### Trabajos Prácticos
Los archivos en `practicos/` son guías de laboratorio:

```
practicos/laboratorio-01.md
```

## 🌐 Desarrollo Local

### Opción 1: Python HTTP Server

```bash
cd docs
python3 -m http.server 8000
```

Accede en: http://localhost:8000

### Opción 2: Node.js http-server

```bash
npm install -g http-server
cd docs
http-server -p 8000
```

### Opción 3: VS Code Live Server

1. Instala la extensión "Live Server"
2. Click derecho en `docs/index.html`
3. Selecciona "Open with Live Server"

## ✏️ Editar Contenido

### Editar Presentaciones

1. Abre el archivo `.md` en `filminas/`
2. Edita el contenido en Markdown
3. Usa `---` para separar slides
4. Commit y push los cambios
5. El sitio se actualiza automáticamente

Ejemplo:
```markdown
# Título del Slide 1

Contenido del primer slide

---

# Título del Slide 2

Contenido del segundo slide

---
```

### Editar la Wiki

1. Edita archivos en `wiki/` localmente
2. Copia el contenido a GitHub Wiki (manual)
3. O edita directamente en GitHub Wiki

### Editar el Portal Principal

Edita `docs/index.html` para modificar:
- Enlaces a presentaciones
- Descripción del curso
- Estructura de navegación

## 🎨 Personalización

### Cambiar Tema de Reveal.js

En `docs/slides.html`, línea del theme:

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@4.5.0/dist/theme/black.css">
```

Temas disponibles: black, white, league, beige, sky, night, serif, simple, solarized

### Personalizar Colores

Edita los estilos CSS en:
- `docs/index.html` (portal principal)
- `docs/wiki.html` (wiki integrada)
- `docs/practicos.html` (trabajos prácticos)

## 🔍 Solución de Problemas

### El sitio no se actualiza

1. Verifica que GitHub Actions se ejecutó correctamente
2. Espera 2-3 minutos para propagación
3. Limpia la caché del navegador (Ctrl+Shift+R)

### Las presentaciones no cargan

1. Verifica que los archivos `.md` existen en `filminas/`
2. Verifica las rutas en `docs/index.html`
3. Revisa la consola del navegador (F12) para errores

### Errores de sintaxis en Markdown

1. Verifica que usas `---` para separar slides
2. Revisa la sintaxis de Markdown
3. Usa un validador de Markdown online

## 📦 Dependencias

El sitio usa CDN para todas las librerías (no requiere instalación local):

- **Reveal.js 4.5.0**: Presentaciones
- **Highlight.js**: Syntax highlighting en código
- No requiere Node.js, npm ni build

## 🚀 Despliegue Automático

Cada vez que haces `git push` a la rama `main`:

1. GitHub Actions se activa automáticamente
2. El contenido de `docs/` se despliega
3. El sitio se actualiza en minutos

## 📝 Checklist de Configuración Inicial

- [x] Repositorio creado
- [ ] GitHub Pages habilitado (Source: GitHub Actions)
- [ ] Primer push realizado
- [ ] Workflow de Actions ejecutado correctamente
- [ ] Sitio accesible en `https://usuario.github.io/ARyS-U7/`
- [ ] GitHub Wiki inicializada
- [ ] Contenido de `wiki/` copiado a GitHub Wiki

## 🤝 Flujo de Trabajo Recomendado

1. **Crear contenido localmente**
   ```bash
   # Editar archivos en filminas/, teoria/, practicos/
   vim filminas/nueva-presentacion.md
   ```

2. **Probar localmente**
   ```bash
   cd docs
   python3 -m http.server 8000
   # Abrir http://localhost:8000
   ```

3. **Commit y push**
   ```bash
   git add .
   git commit -m "Agregar nueva presentación sobre..."
   git push
   ```

4. **Verificar despliegue**
   - Ir a Actions
   - Esperar que complete
   - Verificar en el sitio web

## 📚 Recursos Adicionales

- [Reveal.js Documentation](https://revealjs.com/)
- [GitHub Pages Docs](https://docs.github.com/en/pages)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [Markdown Guide](https://www.markdownguide.org/)

## 💡 Tips

- **Presentaciones**: Usa notas del presentador con `Note:` en los slides
- **Imágenes**: Coloca en `docs/images/` y referencia con rutas relativas
- **Videos**: Usa embeds de YouTube o enlaces externos
- **Código**: Usa syntax highlighting especificando el lenguaje

---

¿Dudas? Abre un [Issue](https://github.com/bzappellini/ARyS-U7/issues) en el repositorio.
