# 📋 Instrucciones de Configuración Post-Merge

## ¡Felicidades! 🎉

Has recibido una infraestructura completa para tus materiales de ARyS-U7. Sigue estos pasos para activarlo todo.

## Paso 1: Habilitar GitHub Pages

1. Ve a tu repositorio en GitHub: `https://github.com/bzappellini/ARyS-U7`
2. Haz clic en **Settings** (⚙️ Configuración)
3. En el menú lateral, haz clic en **Pages**
4. En **Source**, selecciona: **GitHub Actions**
5. Guarda los cambios

### Verificar el Deploy

1. Ve a la pestaña **Actions** en tu repositorio
2. Verás el workflow "Deploy to GitHub Pages" ejecutándose
3. Espera a que termine (aparecerá un ✅ verde)
4. Tu sitio estará disponible en: `https://bzappellini.github.io/ARyS-U7/`

## Paso 2: Configurar GitHub Wiki

### Opción A: Copiar Manualmente

1. Ve a la pestaña **Wiki** en tu repositorio
2. Haz clic en "Create the first page"
3. Copia el contenido de `wiki/Home.md` y pégalo en la página Home
4. Guarda la página
5. Crea páginas adicionales:
   - **Introduccion-AAA**: Copia el contenido de `wiki/Introduccion-AAA.md`
   - Puedes crear más páginas según necesites

### Opción B: Usar Git (Avanzado)

```bash
# Clonar el wiki
git clone https://github.com/bzappellini/ARyS-U7.wiki.git
cd ARyS-U7.wiki

# Copiar archivos
cp ../wiki/*.md .

# Commit y push
git add .
git commit -m "Add wiki content"
git push
```

## Paso 3: Verificar que Todo Funciona

### ✅ Checklist de Verificación

Visita tu sitio web y verifica:

- [ ] El portal principal carga correctamente
- [ ] Las presentaciones se pueden abrir (sin errores de Reveal.js)
- [ ] La wiki integrada muestra el contenido
- [ ] Los trabajos prácticos se muestran correctamente
- [ ] La navegación funciona entre páginas

### URLs a Verificar

```
Portal principal:
https://bzappellini.github.io/ARyS-U7/

Presentaciones:
https://bzappellini.github.io/ARyS-U7/slides.html?md=filminas/introduccion-aaa.md
https://bzappellini.github.io/ARyS-U7/slides.html?md=filminas/protocolos-aaa.md
https://bzappellini.github.io/ARyS-U7/slides.html?md=filminas/implementacion-aaa.md

Wiki integrada:
https://bzappellini.github.io/ARyS-U7/wiki.html

Trabajos prácticos:
https://bzappellini.github.io/ARyS-U7/practicos.html

GitHub Wiki:
https://github.com/bzappellini/ARyS-U7/wiki
```

## Paso 4: Compartir con Estudiantes

Una vez verificado que todo funciona:

1. Comparte la URL principal: `https://bzappellini.github.io/ARyS-U7/`
2. Los estudiantes podrán acceder a:
   - Presentaciones interactivas
   - Material teórico
   - Guías de laboratorio

## 🎨 Personalización

### Cambiar el Tema de las Presentaciones

Edita `docs/slides.html`, línea 9:

```html
<!-- Temas disponibles: black, white, league, beige, sky, night, serif, simple, solarized -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/reveal.js@4.5.0/dist/theme/black.css">
```

### Agregar Nuevas Presentaciones

1. Crea un archivo `.md` en `filminas/`
2. Usa `---` para separar slides
3. Agrega el enlace en `docs/index.html`
4. Commit y push

### Editar Contenido Existente

Todo el contenido está en Markdown:
- Presentaciones: `filminas/*.md`
- Teoría: `teoria/*.md`
- Prácticos: `practicos/*.md`
- Wiki: `wiki/*.md`

Simplemente edita los archivos y haz commit/push. GitHub Actions actualizará el sitio automáticamente.

## 🔧 Solución de Problemas

### El sitio no se actualiza

- Espera 2-3 minutos después del push
- Verifica que el workflow de Actions se ejecutó correctamente
- Limpia la caché del navegador (Ctrl+Shift+R)

### Las presentaciones no cargan (ERR_BLOCKED)

- Esto es normal en entornos de testing
- En GitHub Pages funcionarán correctamente
- Si usas bloqueadores de anuncios, desactívalos temporalmente

### Error 404 en presentaciones

- Verifica que los archivos existen en `docs/filminas/`
- Asegúrate de que GitHub Actions se ejecutó
- Revisa las rutas en `docs/index.html`

## 📞 Soporte

Si tienes problemas:

1. Revisa el archivo `SETUP.md` para más detalles
2. Verifica los logs en GitHub Actions
3. Abre un Issue en el repositorio

## 🎓 Uso en Clase

### Para Presentar

1. Abre la presentación en tu navegador
2. Presiona `F` para pantalla completa
3. Usa las flechas del teclado para navegar
4. Presiona `S` para modo presentador (notas)
5. Presiona `O` para vista general

### Para Estudiantes

Comparte el enlace principal y ellos podrán:
- Ver las presentaciones a su ritmo
- Leer el material teórico
- Descargar las guías de laboratorio
- Consultar la wiki cuando necesiten

## ✨ Características Adicionales

### Modo Presentador (Reveal.js)

- Presiona `S` durante una presentación
- Se abrirá una ventana con:
  - Vista del slide actual
  - Vista del siguiente slide
  - Notas del presentador
  - Temporizador

### Búsqueda en la Wiki

- La wiki tiene navegación lateral
- Usa Ctrl+F para buscar en la página
- Los enlaces internos funcionan con scroll suave

### Responsive Design

- Todo el sitio es responsive
- Funciona en móviles y tablets
- Las presentaciones se adaptan al tamaño de pantalla

## 📚 Próximos Pasos Recomendados

1. ✅ Configura GitHub Pages
2. ✅ Copia contenido a GitHub Wiki
3. ✅ Verifica todas las URLs
4. ✅ Prueba las presentaciones
5. ✅ Comparte con estudiantes
6. 📝 Agrega más contenido según necesites
7. 🎨 Personaliza colores y estilos si lo deseas
8. 📊 Considera agregar más presentaciones

## 🎯 Contenido Actual

### Presentaciones (3)
- ✅ Introducción a AAA
- ✅ Protocolos AAA
- ✅ Implementación de AAA

### Material Teórico
- ✅ Conceptos fundamentales de AAA
- ✅ Comparación de protocolos
- ✅ Casos de uso empresariales
- ✅ Mejores prácticas de seguridad

### Laboratorios (6)
- ✅ Lab 1: Instalación FreeRADIUS (Básico, 60 min)
- ✅ Lab 2: Integración LDAP/AD (Intermedio, 90 min)
- ✅ Lab 3: WiFi 802.1X (Intermedio, 120 min)
- ✅ Lab 4: Accounting (Básico, 60 min)
- ✅ Lab 5: TACACS+ (Avanzado, 120 min)
- ✅ Lab 6: Alta Disponibilidad (Avanzado, 90 min)

---

**¿Todo listo?** ¡Excelente! Tu plataforma educativa está ahora completamente operativa. 🚀

**URL Principal**: https://bzappellini.github.io/ARyS-U7/

_¿Preguntas? Consulta SETUP.md o README.md para más información._
