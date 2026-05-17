# CLAUDE.md — Refactorización Mobile-First: Sitio Minecraft

## Estructura del proyecto

```
minecraft-clone/
├── imagenes/          # imágenes estáticas del sitio
├── index.html         # único archivo HTML
├── style.css          # estilos principales (importa variables.css)
├── variables.css      # variables CSS (light/dark theme)
└── README.md
```

Solo modificar: `style.css`. No tocar `variables.css`, `index.html` ni las imágenes.

---

## Problema actual

El CSS actual está escrito en enfoque **desktop-first**: los estilos base son para desktop y usa `min-width: 850px` para ajustes. Esto causa que en mobile (< 850px) el layout se vea mal. Hay que convertirlo a **mobile-first** usando `min-width` como estrategia principal.

---

## Breakpoints a usar

| Nombre  | `min-width` |
|---------|-------------|
| Mobile  | base (sin media query) |
| Tablet  | `768px` |
| Desktop | `850px` (mantener este valor para no romper lo que ya funciona) |
| Wide    | `1200px` (ya existe) |

---

## Cambios específicos en `style.css`

### 1. `header .container` y `header nav`

**Problema actual:** En mobile el header apila logo y nav en columna, centrado — esto es correcto visualmente pero el header no está `position: fixed` en mobile, lo que hace que desaparezca al hacer scroll.

**Fix:**
- Hacer el header `position: fixed; width: 100%; z-index: 10;` desde el inicio (base mobile), no solo desde `850px`.
- En mobile mantener el layout en columna y centrado (como está), pero asegurarse de que el body tenga `padding-top` suficiente para que el contenido no quede debajo del header fijo. Calcular según la altura del header en mobile (aproximadamente `130px`).

```css
/* BASE (mobile) */
header {
  position: fixed;
  width: 100%;
  z-index: 10;
}
body {
  padding-top: 130px; /* evita que el hero quede tapado por el header fijo */
}
```

En desktop (`min-width: 850px`) el `padding-top` del body puede reducirse o eliminarse si el hero ya lo compensa con `height: 90vh`.

---

### 2. `h1`, `h2`, texto global

**Problema actual:** `h1` tiene `font-size: 3.5em` y `h2` tiene `font-size: 2.7em` — demasiado grande para mobile, causa overflow horizontal.

**Fix:** Reducir tamaños base para mobile y escalar en desktop.

```css
h1 { font-size: 2em; }
h2 { font-size: 1.8em; }

@media (min-width: 850px) {
  h1 { font-size: 3.5em; }
  h2 { font-size: 2.7em; }
}
```

---

### 3. `#hero`

**Problema actual:** El `h1` del hero tiene `font-size: 5em` solo en desktop. En mobile hereda `3.5em` del `h1` global, que es demasiado.

**Fix:** El `font-size` del `#hero h1` en mobile debe ser más pequeño, y escalar a `5em` en desktop (ya está en el media query, solo asegurarse de que el base sea correcto tras el fix del punto anterior).

También agregar `padding: 0 1rem` al `#hero` para que el texto no toque los bordes en mobile.

```css
#hero {
  padding: 0 1rem; /* añadir */
}
```

---

### 4. `#Que-es-minecraft .container`

**Problema actual:** En mobile no se muestra la imagen (`img-container`) porque solo tiene background-image asignado en `min-width: 850px`. En mobile el div queda vacío e invisible.

**Fix:**
- En mobile: layout en columna, imagen arriba con dimensiones explícitas.
- En desktop: layout en fila (ya existe el media query, mantener).

```css
/* BASE (mobile) */
#Que-es-minecraft .container {
  padding: 30px 1.5rem;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}
#Que-es-minecraft .img-container {
  background-image: url("imagenes/personajes.jpg");
  background-size: cover;
  background-position: center center;
  width: 100%;
  max-width: 350px;
  height: 280px;
  border-radius: 10px;
}

/* DESKTOP (min-width: 850px) — mantener como está */
```

---

### 5. `#Nuestros-Juegos`

**Problema actual:** Las `.carta` en mobile tienen `padding: 200px 50px` — demasiado. Y el `.Juegos` no tiene flex, entonces las cartas van en columna pero con tamaños exagerados.

**Fix en mobile:**
- `.Juegos`: `display: flex; flex-direction: column; align-items: center;`
- `.carta`: reducir padding, hacer ancho razonable.
- Mostrar el `<p>` (texto descriptivo) también en mobile (actualmente está `display: none` en la base y solo se muestra en desktop — invertir esto).

```css
/* BASE (mobile) */
#Nuestros-Juegos .container {
  padding: 40px 1rem;
}
#Nuestros-Juegos h2 {
  font-size: 2.2em;
}
#Nuestros-Juegos p {
  display: block; /* visible en mobile */
}
#Nuestros-Juegos .Juegos {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}
#Nuestros-Juegos .carta {
  width: 100%;
  max-width: 400px;
  padding: 180px 20px 20px; /* imagen arriba, texto abajo */
  margin: 0;
  border-radius: 15px;
  background-size: 100% 180px;
  background-repeat: no-repeat;
  background-position-y: 0;
  background-color: var(--bg-carta);
}

/* DESKTOP (min-width: 850px) — mantener flex-direction: row y estilos actuales */
```

---

### 6. `#caracteristicas`

**Problema actual:** El panda GIF de fondo solo aparece en `min-width: 850px`. En mobile el contenedor tiene `padding: 200px 50px` — excesivo.

**Fix en mobile:**
- Reducir padding a algo razonable: `60px 1.5rem`.
- No mostrar el panda como background en mobile (está bien que no aparezca).
- La lista debe estar centrada en mobile (o alineada a la izquierda con padding).

```css
/* BASE (mobile) */
#caracteristicas .container {
  padding: 60px 1.5rem;
  text-align: center;
}
#caracteristicas ul {
  display: inline-block;
  text-align: left;
}

/* DESKTOP (min-width: 850px) — mantener como está */
```

---

### 7. `#final`

**Problema actual:** `font-size: 9vw` para `h2` y `font-size: 5vw` para el botón — en mobile (375px) esto resulta en ~34px y ~19px respectivamente. Están bien en mobile, pero en desktop se sobreescriben correctamente con el media query. Verificar que quede legible.

No requiere cambio mayor, pero agregar `padding: 0 1rem` para evitar que el texto toque bordes.

```css
#final {
  padding: 0 1rem; /* añadir */
}
```

---

### 8. `footer .container`

**Problema actual:** En mobile el footer centra el copyright (bien), pero en desktop se alinea a la derecha. Esto ya funciona correctamente con el media query existente. **No requiere cambios.**

---

## Orden de trabajo sugerido

1. Arreglar `header` + `body padding-top` para que el header fijo no tape contenido en mobile.
2. Reducir `h1` y `h2` globales para mobile.
3. Arreglar `#Que-es-minecraft` para que la imagen aparezca en mobile.
4. Arreglar `.carta` de `#Nuestros-Juegos` para que se vean bien apiladas en mobile.
5. Reducir padding excesivo en `#caracteristicas`.
6. Revisar `#hero` y `#final` con padding lateral.
7. Probar en DevTools a 375px, 390px y 768px.

---

## Restricciones

- **No modificar** `variables.css`, `index.html` ni las imágenes.
- **No instalar** dependencias ni frameworks.
- **No cambiar** la lógica del dark mode (funciona correctamente).
- **Mantener** el breakpoint `850px` para desktop (está bien calibrado).
- **Mantener** el breakpoint `1200px` para wide (ya existe y funciona).
- Usar solo las variables CSS definidas en `variables.css` para colores.

---

## Criterio de éxito

Verificar en DevTools (Device Toolbar) que se vea correctamente en:
- [ ] 375px — iPhone SE
- [ ] 390px — iPhone 14
- [ ] 768px — iPad
- [ ] 1024px — Desktop
- [ ] 1440px — Desktop wide

En ninguna resolución debe haber scroll horizontal ni texto desbordado.
