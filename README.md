# 🎨 canaima-titlebar (`lib-rust-titlebar`)

<div align="center">

[![Canaima GNU/Linux](https://img.shields.io/badge/Canaima-GNU%2FLinux-0B6793?style=for-the-badge&logo=linux&logoColor=white)](https://canaima.softwarelibre.gob.ve/)
[![Rust 2021](https://img.shields.io/badge/Rust-2021%20Edition-black?style=for-the-badge&logo=rust)](https://www.rust-lang.org/)
[![GTK 3](https://img.shields.io/badge/GTK-3.24-4B85BE?style=for-the-badge&logo=gtk&logoColor=white)](https://www.gtk.org/)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg?style=for-the-badge)](LICENSE)
[![Tauri v2](https://img.shields.io/badge/Tauri-v2.x-24C8DB?style=for-the-badge&logo=tauri&logoColor=white)](https://tauri.app/)

**Barra de título nativa GTK HeaderBar con estética e identidad visual de Canaima GNU/Linux para aplicaciones Tauri v2.**

</div>

---

## 📖 Descripción General

**`canaima-titlebar`** proporciona una cabecera de ventana nativa (`GtkHeaderBar`) adaptada para aplicaciones desarrolladas con **Tauri 2** en Canaima GNU/Linux y otras distribuciones basadas en Debian.

Integra la identidad visual oficial del sistema operativo Canaima, incluyendo el isotipo vectorial del **Tepuy**, controles de ventana circulares y soporte completo para cambio en caliente entre tema claro y tema oscuro desde el frontend (Vue 3, React, Svelte o JavaScript puro).

---

## ✨ Características Principales

- ⛰️ **Isotipo Oficial Canaima Vectorial**: Incorpora el logo Tepuy en formato SVG embebido en el binario; se adapta y recolorea dinámicamente al alternar entre modo oscuro y claro sin pérdida de nitidez.
- 🔴🟡🔵 **Botones de Control Estilizados**: Botones circulares con la paleta de color oficial de Canaima (azul `#0B6793`, gris `#d6d6d6` y `#303030`), estados `:hover` y `:backdrop`.
- 🔀 **Cero Archivos Externos**: Todos los iconos y gráficos están codificados directamente en Data URIs Base64 y SVG embebido. Tu aplicación no dependerá de carpetas externas de `assets/` ni fallará al ejecutarse desde `/usr/bin` o lanzadores `.desktop`.
- 🌗 **Sincronización Dinámica de Tema**: Expone el comando Tauri `sync_theme` para conmutar la barra entre tema claro y oscuro en tiempo real cuando el usuario cambie el tema en la interfaz web.
- 🖐️ **Interacción de Ventana Fluida**:
  - Arrastre interactivo de ventana mediante `begin_move_drag`.
  - Doble clic en la barra para maximizar y restaurar.
- 🛡️ **Protección Anti-Marcos Negros en X11**: Detecta automáticamente si el servidor gráfico no tiene compositor activo (`!screen.is_composited()`, típico en **LXDE**, **LXQt** con Openbox o **XFCE/MATE** sin composición) y aplica reglas `.solid-csd` sin sombras transparentes para evitar rectángulos negros alrededor de la ventana.
- 🧹 **Gestión de Memoria y Rendimiento**: Carga única de estilos en el `GdkScreen` mediante `std::sync::Once` evitando fugas de memoria o proveedores CSS redundantes.

---

## 🖥️ Compatibilidad de Entornos de Escritorio

| Entorno | Gestor de Ventanas | Decoraciones | Soporte |
| :--- | :--- | :--- | :---: |
| **GNOME** | Mutter | CSD nativo con sombra RGBA | ✅ 100% |
| **KDE Plasma** | KWin | CSD nativo con integración KWin | ✅ 100% |
| **XFCE** | Xfwm4 | CSD / `_GTK_FRAME_EXTENTS` | ✅ 100% |
| **LXDE** | Openbox | Modo `solid-csd` sin marcos negros | ✅ 100% |
| **MATE** | Marco | CSD adaptativo con y sin compositor | ✅ 100% |
| **Cinnamon** | Muffin | CSD nativo con aceleración | ✅ 100% |
| **LXQt** | Openbox / KWin | Integración fluida en Wayland y X11 | ✅ 100% |

---

## 📦 Instalación

Agrega `canaima-titlebar` a tu archivo `src-tauri/Cargo.toml`:

```toml
[dependencies]
canaima-titlebar = { path = "ruta/a/lib-rust-titlebar-main" }
```

O desde Git:

```toml
[dependencies]
canaima-titlebar = { git = "https://github.com/SistemaOperativoCanaima/rust-titlebar-canaima.git", branch = "main" }
```

---

## 💻 Guía de Integración

### 1. Configuración en Tauri (Rust)

En tu archivo `src-tauri/src/lib.rs`:

```rust
use tauri::Manager;

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .setup(|app| {
            #[cfg(target_os = "linux")]
            {
                // Obtener la ventana principal e inicializar la HeaderBar Canaima
                if let Some(window) = app.get_webview_window("main") {
                    canaima_titlebar::setup_default(&window);
                }
            }
            Ok(())
        })
        // Registrar el comando Tauri para sincronización de tema
        .invoke_handler(tauri::generate_handler![
            canaima_titlebar::sync_theme
        ])
        .run(tauri::generate_context!())
        .expect("Error al ejecutar la aplicación");
}
```

---

### 2. Configuración Personalizada (`TitlebarConfig`)

Si deseas cambiar el título, ocultar algún botón o añadir CSS personalizado:

```rust
use canaima_titlebar::{setup_gtk_headerbar, TitlebarConfig};

if let Some(window) = app.get_webview_window("main") {
    let config = TitlebarConfig {
        enabled: true,
        title: Some("Mi Aplicación Canaima".to_string()),
        show_logo: true,           // Mostrar isotipo Tepuy
        show_minimize: true,       // Botón minimizar
        show_maximize: true,       // Botón maximizar
        show_close: true,          // Botón cerrar
        initial_dark_mode: false,  // Iniciar en modo claro
        custom_css: Some("
            #canaima-headerbar {
                background-color: #f5f5f5;
            }
        ".to_string()),
    };

    setup_gtk_headerbar(&window, &config);
}
```

---

### 3. Sincronización desde el Frontend (JavaScript / TypeScript / Vue 3)

Cuando tu usuario cambie el tema en la aplicación web, invoca `sync_theme` para actualizar instantáneamente la barra nativa GTK:

```typescript
import { invoke } from '@tauri-apps/api/core';

// Alternar entre modo oscuro y claro
export async function cambiarTema(esOscuro: boolean) {
  // 1. Actualizar clases en el frontend
  if (esOscuro) {
    document.documentElement.classList.add('dark');
  } else {
    document.documentElement.classList.remove('dark');
  }

  // 2. Notificar a la HeaderBar nativa de Canaima
  await invoke('sync_theme', { isDark: esOscuro });
}
```

---

## 🛠️ Personalización con CSS

Los elementos de la barra cuentan con identificadores y clases semánticas para facilitar estilos personalizados:

| Selector | Descripción |
| :--- | :--- |
| `#canaima-headerbar` | Contenedor principal de la HeaderBar. |
| `#canaima-headerbar.is-dark` | Estilos aplicados en modo oscuro. |
| `#canaima-title` | Etiqueta de texto del título centrado. |
| `#canaima-logo` | Widget de imagen con el logo SVG vectorial de Canaima. |
| `#canaima-buttons-box` | Contenedor de los botones de ventana. |
| `.btn-minimize` | Botón circular de minimizar. |
| `.btn-maximize` | Botón circular de maximizar / restaurar. |
| `.btn-close` | Botón circular de cerrar (color distintivo `#0B6793`). |

---

## 🧪 Pruebas Unitarias

Para validar la integridad de la librería y de los activos embebidos:

```bash
cargo test
```

Para verificar estándares de calidad con Clippy:

```bash
cargo clippy -- -D warnings
```

---

## 👥 Equipo y Créditos

Desarrollado con dedicación por el **Equipo de Desarrollo de Canaima GNU/Linux**.

- **Sitio Oficial**: [https://canaima.softwarelibre.gob.ve/](https://canaima.softwarelibre.gob.ve/)
- **Repositorio**: [GitHub Sistema Operativo Canaima](https://github.com/SistemaOperativoCanaima)

---

## 📄 Licencia

Este proyecto se distribuye bajo la licencia **GNU General Public License v3.0 (GPL-3.0-only)**. Consulta el archivo `LICENSE` para más detalles.
