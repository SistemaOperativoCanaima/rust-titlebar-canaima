# 🎨 canaima-titlebar (`lib-rust-titlebar`)

<div align="center">

[![Canaima GNU/Linux](https://img.shields.io/badge/Canaima-GNU%2FLinux-0B6793?style=for-the-badge&logo=linux&logoColor=white)](https://canaima.softwarelibre.gob.ve/)
[![Rust 2021](https://img.shields.io/badge/Rust-2021%20Edition-black?style=for-the-badge&logo=rust)](https://www.rust-lang.org/)
[![GTK 3](https://img.shields.io/badge/GTK-3.24-4B85BE?style=for-the-badge&logo=gtk&logoColor=white)](https://www.gtk.org/)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg?style=for-the-badge)](LICENSE)
[![Tauri v2](https://img.shields.io/badge/Tauri-v2.x-24C8DB?style=for-the-badge&logo=tauri&logoColor=white)](https://tauri.app/)

**Barra de título nativa GTK HeaderBar con identidad visual oficial de Canaima GNU/Linux para aplicaciones Tauri v2.**

</div>

---

## 📖 Descripción General

**`canaima-titlebar`** proporciona una cabecera de ventana nativa (`GtkHeaderBar`) adaptada para aplicaciones desarrolladas con **Tauri 2** en Canaima GNU/Linux y sistemas basados en Debian.

Sustituye por completo las barras de título genéricas del sistema operativo por la **identidad visual oficial de Canaima**, incorporando el isotipo vectorial del **Tepuy**, controles de ventana circulares con la paleta de colores institucional, arrastre interactivo nativo y sincronización en tiempo real con el modo claro/oscuro del frontend.

---

## ✨ Características Principales

- ⛰️ **Isotipo Vectorial Oficial Canaima**: Logo Tepuy en formato SVG embebido directamente en el binario; calibrado a **71 × 12 px** y recoloreado dinámicamente según el tema activo sin pérdida de nitidez.
- 🔴🟡🔵 **Controles de Ventana Calibrados Milimétricamente**:
  - Botones circulares con diámetro exacto de **20 × 20 px** y margen exterior derecho de **12 px**.
  - Separación uniforme de **7 px** entre controles.
  - Iconos institucionales de **8 × 8 px** renderizados con `PixbufLoader` nativo para máxima nitidez en pantallas estándar y HiDPI:
    - **Cerrar ($\times$)**: Cruz delgada estilizada de 1 px de trazo con antialiasing simétrico 8-way sobre fondo azul Canaima (`#0B6793` y `:hover` `#0889c7`).
    - **Maximizar ($\square$)**: Marco cuadrado fino de 1 px, esquinas suavizadas y centro hueco.
    - **Minimizar (—)**: Barra horizontal nítida de 1 px alineada en la línea base de los controles.
- 🪟 **Soporte Nativo de Segundo Plano (`:backdrop`)**:
  - Cuando la ventana pasa a segundo plano o se sitúa por debajo de otra, los botones de control, el título y el isotipo se atenúan automáticamente al **55% de opacidad** (`opacity: 0.55`).
  - Transición fluida con `transition: opacity 0.2s ease` para otorgar la jerarquía visual nativa del sistema operativo sin saltos abruptos.
- 🌓 **Detección Automática y Sincronización de Tema**:
  - Detecta automáticamente el tema del escritorio en el arranque mediante `detect_system_dark_theme()`.
  - Expone el comando Tauri `sync_theme` para conmutar la barra instantáneamente cuando el usuario cambie el tema en la interfaz web (Vue, React, Svelte, Tailwind).
- 🛡️ **Protección Anti-Bordes Negros en X11**:
  - Detecta si el servidor de visualización carece de compositor activo (`!screen.is_composited()`, común en **LXDE**, **LXQt** con Openbox o **XFCE/MATE** sin composición).
  - Aplica automáticamente la clase `.solid-csd` con bordes sólidos y sin sombras transparentes RGBA, eliminando por completo los rectángulos negros alrededor de la ventana.
- 🖐️ **Comportamiento Nativo Completo**:
  - Arrastre interactivo de ventana mediante `begin_move_drag`.
  - Doble clic en la barra para alternar entre maximizar y restaurar.
- 🧹 **Cero Archivos Externos y Carga Única**:
  - Todos los activos (SVG e imágenes) están compilados dentro del binario. Tu aplicación no depende de carpetas externas de `assets/` ni fallará al empaquetarse en `.deb`.
  - Inyección global de estilos CSS en `GdkScreen` gestionada con `std::sync::Once` para evitar duplicación y fugas de memoria.

---

## 🖥️ Matriz de Compatibilidad por Entorno de Escritorio

| Entorno | Gestor de Ventanas | Decoraciones | Soporte |
| :--- | :--- | :--- | :---: |
| **GNOME** | Mutter | CSD nativo con sombra RGBA suave | ✅ 100% |
| **KDE Plasma** | KWin | CSD nativo con integración KWin | ✅ 100% |
| **XFCE** | Xfwm4 | CSD / `_GTK_FRAME_EXTENTS` nativos | ✅ 100% |
| **LXDE** | Openbox | Modo `solid-csd` sin marcos negros | ✅ 100% |
| **MATE** | Marco | CSD adaptable con y sin composición | ✅ 100% |
| **Cinnamon** | Muffin | CSD nativo con aceleración | ✅ 100% |
| **LXQt** | Openbox / KWin | Integración fluida en Wayland y X11 | ✅ 100% |

---

## 🚀 Guía de Integración Paso a Paso: ¿Cómo hacerlo funcionar en tu App?

Para que tu aplicación de escritorio muestre la ventana personalizada de Canaima **sin que aparezca primero la ventana del sistema operativo**, el desarrollador debe seguir estos pasos en su proyecto Tauri v2:

---

### Paso 1: Configurar `tauri.conf.json` (¡CRUCIAL!)

En tu archivo `src-tauri/tauri.conf.json`, define la configuración de la ventana principal:

```json
{
  "app": {
    "windows": [
      {
        "title": "Mi Aplicación Canaima",
        "width": 800,
        "height": 600,
        "minWidth": 400,
        "minHeight": 300,
        "decorations": true,
        "transparent": false,
        "visible": false
      }
    ]
  }
}
```

> [!IMPORTANT]
> **¿Por qué `"visible": false` es obligatorio?**
> En GTK 3, la función [`gtk_window_set_titlebar`](https://docs.gtk.org/gtk3/method.Window.set_titlebar.html) (que asocia la `HeaderBar` nativa de Canaima) **únicamente puede ejecutarse sobre una ventana que aún no ha sido mapeada ni mostrada en pantalla**.
> 
> - Si configuras `"visible": true`, Tauri forzará al sistema operativo a mostrar la ventana inmediatamente con el marco por defecto del gestor de ventanas (Mutter en GNOME, KWin en KDE, Xfwm4, etc.). Esto causará un parpadeo desagradable de la ventana del SO o provocará que GTK ignore la barra personalizada.
> - Con `"visible": false`, la ventana se crea en segundo plano; Rust le adjunta la `HeaderBar` de Canaima limpiamente, y la ventana sólo se muestra cuando la interfaz web (Vue / HTML) ha terminado de cargar, logrando una presentación impecable.
> 
> **¿Por qué `"decorations": true`?**
> En GTK, si colocas `"decorations": false`, GTK desactiva completamente el sistema de decoración CSD y ocultará tanto la barra nativa del SO como la propia `HeaderBar`.

---

### Paso 2: Otorgar Permisos en `capabilities/default.json`

Para que el frontend de tu aplicación (TypeScript / JavaScript) tenga autorización para mostrar la ventana una vez cargada, verifica que `core:window:allow-show` esté presente en `src-tauri/capabilities/default.json`:

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "default",
  "description": "Permisos de ventana para la plantilla Canaima",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "core:window:allow-show",
    "core:window:allow-minimize",
    "core:window:allow-maximize",
    "core:window:allow-close",
    "core:window:allow-toggle-maximize",
    "core:window:allow-start-dragging",
    "opener:default"
  ]
}
```

---

### Paso 3: Agregar Dependencias en `src-tauri/Cargo.toml`

Agrega `canaima-titlebar` junto a las librerías GTK de Linux:

```toml
[dependencies]
tauri = { version = "2" }
canaima-titlebar = { path = "../../../lib-rust-titlebar-main" }

[target.'cfg(target_os = "linux")'.dependencies]
gtk = "0.18"
glib = "0.18"
gdk-pixbuf = "0.18"
```

---

### Paso 4: Inicializar la Barra y Registrar Comandos en `src-tauri/src/lib.rs`

En la función de inicialización de Tauri, asocia la barra a la ventana principal dentro de `.setup()` y registra el comando `canaima_titlebar::sync_theme`:

```rust
// src-tauri/src/lib.rs

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_opener::init())
        .setup(|app| {
            #[cfg(target_os = "linux")]
            {
                use tauri::Manager;
                // Obtener la ventana principal antes de mapearla
                if let Some(window) = app.get_webview_window("main") {
                    // Inicializar la HeaderBar Canaima con la configuración oficial
                    canaima_titlebar::setup_default(&window);
                }
            }
            Ok(())
        })
        .invoke_handler(tauri::generate_handler![
            // Registrar comando IPC para cambio dinámico de tema
            canaima_titlebar::sync_theme,
        ])
        .run(tauri::generate_context!())
        .expect("Ocurrió un error al ejecutar la aplicación Canaima");
}
```

---

### Paso 5: Mostrar la Ventana de Forma Segura en el Frontend (`src/main.ts`)

Dado que la ventana se configuró con `"visible": false`, el frontend debe invocar `getCurrentWindow().show()` una vez que el DOM esté listo.

En aplicaciones que usan empaquetadores como **Vite** (`<script type="module">`), los scripts se ejecutan de forma diferida. Para evitar condiciones de carrera donde `DOMContentLoaded` ya se haya emitido antes de registrar el listener, usa el siguiente patrón seguro en `src/main.ts`:

```typescript
// src/main.ts
import { getCurrentWindow } from '@tauri-apps/api/window'
import { createApp } from 'vue'
import App from './App.vue'

// 1. Mostrar la ventana de forma segura (evita pantalla blanca y ventana del SO)
const showWindow = () => {
  getCurrentWindow().show().catch(() => null)
}

if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', showWindow)
} else {
  // Si el DOM ya cargó antes de ejecutar el módulo Vite, mostrar de inmediato
  showWindow()
}

// 2. Montar tu aplicación Vue / React
const app = createApp(App)
app.mount('#app')
```

---

### Paso 6: Sincronización de Tema Claro / Oscuro desde el Frontend

Cuando el usuario alterne el tema de tu aplicación (por ejemplo, desde un botón en la interfaz o mediante un Store de Pinia), invoca `sync_theme` para sincronizar la HeaderBar nativa:

```typescript
// src/stores/useAppStore.ts (o en cualquier componente Vue / React)
import { invoke } from '@tauri-apps/api/core'

export function aplicarTema(esOscuro: boolean) {
  // 1. Actualizar clases CSS en el DOM web (Tailwind CSS)
  if (esOscuro) {
    document.documentElement.classList.add('dark')
  } else {
    document.documentElement.classList.remove('dark')
  }

  // 2. Notificar a la HeaderBar nativa GTK
  invoke('sync_theme', { isDark: esOscuro }).catch((err) => {
    console.error('Error al sincronizar tema con canaima-titlebar:', err)
  })
}
```

---

## 🛠️ Personalización Flexible: Solo las opciones que necesitas

A partir de la versión `0.1.1`, **ya no es necesario especificar todas las variables ni campos de la estructura**. Puedes modificar únicamente la opción deseada utilizando el patrón *Builder* o las funciones de conveniencia directa.

### 🌟 Opción A: Modo Plugin Tauri (Recomendado - 1 sola línea)

Configura la ventana principal (`"main"`) y registra el comando `sync_theme` automáticamente sin escribir código en `setup()`:

```rust
// Opciones por defecto oficiales
tauri::Builder::default()
    .plugin(canaima_titlebar::init())
```

O si solo deseas cambiar una opción (por ejemplo, desactivar el botón maximizar en una ventana fija o diálogo):

```rust
use canaima_titlebar::TitlebarConfig;

tauri::Builder::default()
    .plugin(canaima_titlebar::init_with_config(
        TitlebarConfig::new().without_maximize()
    ))
```

---

### 🌟 Opción B: Mediante `setup` con Builder Fluent

La función `canaima_titlebar::setup(&window, ...)` acepta cualquier configuración gracias al trait `IntoTitlebarConfig`. Puedes pasar:

```rust
use canaima_titlebar::{setup, TitlebarConfig};

// 1. Solo cambiar el título (resto por defecto):
setup(&window, "Centro de Software Canaima");

// 2. Solo desactivar el botón de maximizar (ideal para ventanas de diálogo):
setup(&window, TitlebarConfig::new().without_maximize());

// 3. Solo ocultar el logotipo del Tepuy:
setup(&window, TitlebarConfig::new().without_logo());

// 4. Encadenar múltiples opciones deseadas:
setup(&window, TitlebarConfig::new()
    .with_title("Configuración")
    .without_maximize()
    .with_dark_mode(true)
);

// 5. Opciones por defecto:
setup(&window, ());
```

---

### 🌟 Opción C: Funciones Rápidas Directas

Si solo quieres realizar una acción común en una sola llamada:

```rust
// Solo cambiar el título:
canaima_titlebar::setup_with_title(&window, "Mi Aplicación");

// Solo desactivar maximizar (frecuente en utilitarios o diálogos):
canaima_titlebar::setup_without_maximize(&window);

// Solo desactivar minimizar:
canaima_titlebar::setup_without_minimize(&window);

// Solo ocultar el logo Canaima:
canaima_titlebar::setup_without_logo(&window);

// Forzar modo oscuro inicial:
canaima_titlebar::setup_with_dark_mode(&window, true);
```

---

### 🌟 Opción D: Modificación de campos clásicos (Compatible hacia atrás)

Si prefieres la estructura completa clásica o necesitas inyectar CSS avanzado personalizado:

```rust
use canaima_titlebar::{setup_gtk_headerbar, TitlebarConfig};

let config = TitlebarConfig {
    custom_css: Some("
        #canaima-headerbar {
            background-color: #202020;
        }
    ".to_string()),
    ..TitlebarConfig::default() // Completa automáticamente el resto de valores
};

setup_gtk_headerbar(&window, &config);
```

---

## 🪟 Comportamiento de Ventana Inactiva (`:backdrop`)

Cuando el usuario hace clic fuera de la aplicación o abre otra ventana por encima, GTK activa de forma nativa la pseudo-clase CSS `:backdrop`.

`canaima-titlebar` gestiona este estado automáticamente:
- **Atenuación institucional:** Los tres botones de control (`btn-minimize`, `btn-maximize`, `btn-close`), el título central (`#canaima-title`) y el isotipo (`#canaima-logo`) reducen su opacidad al **55%** (`opacity: 0.55`).
- **Tono azul desaturado:** El botón de cierre `#0B6793` adquiere el tono atenuado característico de Canaima (`rgba(11, 103, 147, 0.55)`), permitiendo al usuario identificar instantáneamente cuál ventana posee el foco del teclado y mouse.
- **Transición suave:** La propiedad `transition: opacity 0.2s ease` garantiza una alternancia visual limpia y profesional.

---

## 🎨 Selectores CSS para Estilos Personalizados

Todos los componentes de la barra exponen selectores semánticos que puedes manipular con la opción `custom_css`:

| Selector | Elemento GTK | Descripción |
| :--- | :--- | :--- |
| `#canaima-headerbar` | `GtkHeaderBar` | Contenedor principal de la barra de título. |
| `#canaima-headerbar.is-dark` | `GtkHeaderBar` | Estilo activo cuando la barra está en modo oscuro. |
| `#canaima-headerbar:backdrop` | `GtkHeaderBar` | Estilo cuando la ventana está en segundo plano / inactiva. |
| `#canaima-title` | `GtkLabel` | Etiqueta con el texto del título central. |
| `#canaima-title:backdrop` | `GtkLabel` | Título atenuado cuando la ventana pierde el foco. |
| `#canaima-logo` | `GtkImage` | Imagen que contiene el SVG oficial del Tepuy Canaima. |
| `#canaima-buttons-box` | `GtkBox` | Contenedor horizontal que agrupa los botones de control. |
| `.btn-minimize` | `GtkButton` | Botón circular de minimizar ventana. |
| `.btn-maximize` | `GtkButton` | Botón circular de maximizar / restaurar ventana. |
| `.btn-close` | `GtkButton` | Botón circular de cerrar ventana (`#0B6793`). |
| `.control-btn:backdrop` | `GtkButton` | Botones de control opacados en estado desenfocado. |
| `.solid-csd` | `GtkWindow` | Clase aplicada automáticamente en escritorios X11 sin compositor. |

---

## 🧪 Pruebas Unitarias

Para validar la suite de pruebas y la integridad de los activos embebidos:

```bash
cargo test
```

Para verificar cumplimiento estricto con Clippy:

```bash
cargo clippy -- -D warnings
```

---

## 👥 Equipo y Créditos

Desarrollado con dedicación por el **Equipo de Desarrollo de Canaima GNU/Linux**.

- **Sitio Web Oficial**: [https://canaima.softwarelibre.gob.ve/](https://canaima.softwarelibre.gob.ve/)
- **Organización en GitHub**: [Sistema Operativo Canaima](https://github.com/SistemaOperativoCanaima)

---

## 📄 Licencia

Este proyecto se distribuye bajo la licencia **GNU General Public License v3.0 (GPL-3.0-only)**. Consulta el archivo `LICENSE` para más información.
