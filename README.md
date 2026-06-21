# ⭐ Mis Estrellas — Tracker de hábitos

App sencilla (un único archivo HTML) para gamificar los hábitos diarios de los peques con un sistema de estrellas. Pensada para que un niño/a de ~7 años la entienda y la disfrute.

## Características
- **Tareas buenas** (suman ⭐) y **cosas a mejorar** (restan), con peso de puntos configurable.
- **Puntuación visual del día** (cara + número) y **gráfico** de los últimos 14 días.
- **Objetivo semanal con premio**: por estrellas totales o por días buenos (con mínimo por día).
- **Niveles** y **racha** de días buenos para motivar.
- **Varios niños** (cada uno con sus tareas, color y estrellas).
- **Guardado local** (localStorage) + **export/import** de copia.
- **Sincronización entre dispositivos** opcional con un solo *código* (vía [JSONBin](https://jsonbin.io), sin servidor propio).
- **PIN de papás** opcional para proteger los ajustes.

## Uso
Abre `index.html` en cualquier navegador (funciona incluso con doble clic, sin servidor).

### Alojado (para abrir desde cualquier dispositivo)
- Vía raw.githack:
  https://raw.githack.com/vtellez/kidstracker/main/index.html
- O activando **GitHub Pages** (Settings → Pages → rama `main`).

## Sincronizar entre dispositivos
1. Crea una cuenta gratis en [jsonbin.io](https://jsonbin.io) y copia tu **Master Key** (API KEYS).
2. En la app: botón **☁️ Nube** → pega la clave → **Activar**. Te dará un **código**.
3. En los demás dispositivos: **☁️ Nube** → pega ese **código** → **Conectar**.

> Cualquiera con el código puede ver/editar los datos; usa una cuenta de JSONBin dedicada si te preocupa.

## Privacidad
No hay backend propio. Los datos viven en el navegador (localStorage) y, si activas la sincronización, en tu cuenta de JSONBin.
