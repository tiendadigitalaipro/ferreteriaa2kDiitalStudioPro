# 🧠 CLAUDE.md — Agente Especialista: Ferretería Zync v1.0 (A2K Digital Studio)

> Lee este archivo antes de cualquier tarea. Evita leer las 1990 líneas completas.

---

## 🎯 ROL DEL AGENTE

Eres el **Agente Técnico Senior** de Ferretería Zync v1.0, un sistema POS para ferreterías venezolanas. Archivo único: `ferreteria-zync.html` (~141 KB, ~1990 líneas, todo inline CSS+HTML+JS).

---

## ⚙️ ARQUITECTURA

```
ferreteria-zync/
├── ferreteria-zync.html   ← App completa (1990 líneas) — ARCHIVO ÚNICO
├── ABRIR-FERRETERIA.bat   ← Abre la app con doble clic
└── CLAUDE.md              ← Este archivo
```

### Estructura interna de `ferreteria-zync.html`

| Bloque | Líneas aprox. | Descripción |
|---|---|---|
| `<head>` + CSS global | 1 – 245 | Variables, estilos completos, responsive |
| HTML Login + App Shell | 246 – 435 | `#loginScreen`, `#demoBlock`, sidebar, topbar |
| HTML Páginas | 310 – 630 | 10 páginas: dashboard, pos, inventario, historial, cotizaciones, caja, clientes, proveedores, reportes, configuración |
| HTML Modales | 631 – 797 | modalProducto, modalAjuste, modalCobro, modalVenta, modalCliente, modalFiado, modalProveedor, modalDeudaProv, modalCotizacion, modalLogout |
| Print areas | 798 – 799 | `#receipt-content`, `#caja-declaracion`, `#cot-print-area` |
| Script 1: DB+DEVICE | 800 – 857 | DB helper (localStorage), getDeviceId(), MASTER_LICS, CATS |
| Script 2: PRODS_DEFAULT | 858 – 947 | 80 productos, 10 categorías |
| Script 3: Iron Lock | 948 – 985 | `_fnv`, `_b36`, `_verificarFirmada`, `validarLicencia` |
| Script 4: Helpers + PAY | 986 – 1023 | fmt$, fmtBs, showToast, openModal, toggleSidebar, PAY_METHODS |
| Script 5: Navigation | 1024 – 1065 | showPage, updateBadges |
| Script 6: Login/Session | 1066 – 1108 | doLogin, doLogout, checkDemoValid, updateDemoBadge |
| Script 7: initApp | 1109 – 1150 | initLock, initApp, window.onload |
| Script 8: Dashboard | 1151 – 1184 | renderDashboard |
| Script 9: POS | 1185 – 1285 | cart, renderPos, addToCart, recalcCart, abrirCobro |
| Script 10: confirmarVenta | 1286 – 1327 | confirmarVenta, imprimirRecibo |
| Script 11: Inventario | 1328 – 1440 | renderInventario, abrirModalProducto, guardarProducto, exportarCSV |
| Script 12: Historial | 1441 – 1485 | renderHistorial, verVenta |
| Script 13-14: Cotizaciones | 1486 – 1590 | renderCotizaciones, guardarCotizacion, imprimirCotizacion |
| Script 15: Caja | 1591 – 1645 | renderCaja, abrirCaja, cerrarCaja, calcDifCaja |
| Script 16: Clientes | 1646 – 1726 | renderClientes, guardarCliente, verFiado, registrarPagoFiado |
| Script 17: Proveedores | 1727 – 1800 | renderProveedores, guardarProveedor, verDeudaProv |
| Script 18: Reportes | 1801 – 1842 | renderReportes |
| Script 19: Configuración | 1843 – 1892 | renderConfig, saveConfig, renderPayMethodsCfg |
| Script 20: Admin Panel | 1893 – 1956 | generarLicencia, generarLicenciaDemo, renderLicTable |
| Script 21: Backup | 1957 – 1987 | exportarBackup, importarBackup, resetVentas, resetTodo |

---

## 🔒 REGLAS DE ORO

### 1. 🔐 IRON LOCK (Script 3, líneas ~948–985)
- `_SIGN_SALT = 'FZYNC2026'`
- Verifica checksum ANTES que device (lección de Frutería Pro)
- Códigos MASTER: `FZ-ADMIN-2026`, `FZ-PRO-ZYNC`, `FZ-DEMO-2026`
- DEMO dura **exactamente 5 días** — `432000000 ms`

### 2. 🗄️ localStorage — prefijo `fz_`
| Clave | Contenido |
|---|---|
| `fz_session` | `{shop, code, tipo, vence, loginAt}` |
| `fz_productos` | Array de productos |
| `fz_ventas` | Array de ventas |
| `fz_cotizaciones` | Array de cotizaciones |
| `fz_clientes` | Array de clientes (con fiado y movsFiado) |
| `fz_proveedores` | Array de proveedores (con deuda y movs) |
| `fz_config` | `{nombre, tasa, dir, tel, rif}` |
| `fz_pay_methods` | Array `{id, active}` |
| `fz_caja_apertura` | Objeto apertura activa o null |
| `fz_caja_cierres` | Array histórico cierres |
| `fz_licencias_generadas` | Array de licencias generadas por admin |
| `fz_demo_activated_[btoa(code)]` | `btoa({ts, h, dev})` — activación DEMO |

### 3. 🎨 DISEÑO
- Color principal: `--accent: #f97316` (naranja industrial)
- Fondo dark: `--bg: #0a0a0f`
- Fuentes: Syne (display), Inter (body), JetBrains Mono (mono)
- **No simplificar ni cambiar el esquema de colores**

---

## 📱 PÁGINAS

| ID | Módulo |
|---|---|
| `#page-dashboard` | KPIs + gráfico 7 días + top 5 + alertas |
| `#page-pos` | POS con carrito, filtro por categoría |
| `#page-inventario` | Tabla con CRUD + ajuste stock |
| `#page-historial` | Ventas con filtros fecha/método |
| `#page-cotizaciones` | Cotizaciones con conversión a venta |
| `#page-caja` | Apertura/cierre de turno |
| `#page-clientes` | Clientes con fiado (crédito) |
| `#page-proveedores` | Proveedores con deuda |
| `#page-reportes` | Estadísticas por período |
| `#page-configuracion` | Config + Admin licencias (solo ADMIN) |

---

## 🔑 FUNCIONES CLAVE

| Función | Línea aprox. | Descripción |
|---|---|---|
| `initLock()` | ~1109 | Verifica sesión al cargar |
| `doLogin()` | ~1066 | Valida licencia y crea sesión |
| `showPage(page, el)` | ~1024 | Navegación entre páginas |
| `renderDashboard()` | ~1151 | KPIs y gráficos |
| `confirmarVenta()` | ~1286 | Procesa venta, descuenta stock |
| `abrirCobro()` | ~1253 | Abre modal de cobro |
| `renderInventario()` | ~1328 | Lista productos con filtros |
| `guardarCotizacion()` | ~1539 | Guarda cotización |
| `convertirCotizacion(id)` | ~1556 | Convierte cotización en venta |
| `abrirCaja()` | ~1616 | Abre turno de caja |
| `cerrarCaja()` | ~1633 | Cierra turno con resumen |
| `generarLicencia()` | ~1916 | Genera código PRO (solo ADMIN) |
| `generarLicenciaDemo()` | ~1928 | Genera código DEMO (solo ADMIN) |

---

## 💳 MÉTODOS DE PAGO (11)

`efectivo_usd`, `efectivo_bs`, `pago_movil`, `zelle`, `binance`, `transferencia`, `debito`, `credito`, `divisas`, `fiado`, `mixto`

---

## 📤 REPOSITORIO GITHUB

- **Repo:** `tiendadigitalaipro/ferreteriaa2kDiitalStudioPro`
- **Ruta local:** `C:\Users\ASUS\Desktop\ferreteria-zync\`

```bash
cd "C:\Users\ASUS\Desktop\ferreteria-zync"
git add ferreteria-zync.html
git commit -m "descripción del cambio"
git push origin main
```

---

## ⚠️ INSTRUCCIONES PARA CLAUDE

1. Lee este CLAUDE.md antes de abrir el HTML completo
2. Para buscar funciones: `Grep` el nombre de la función en el archivo
3. Para editar: lee solo el rango de líneas relevante (ver tabla arriba)
4. El Admin Panel está **embebido en `#page-configuracion`**, no es página separada
5. `abrirModalProducto(null)` acepta null para nuevo producto o id para editar
6. La función `exportarCSV` acepta `'inventario'` o `'historial'` como argumento

---

*Ferretería Zync v1.0 · A2K Digital Studio · 2026*
