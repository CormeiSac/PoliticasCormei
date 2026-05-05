[01-funcionalidades.md](https://github.com/user-attachments/files/27412305/01-funcionalidades.md)
# Checklist de funcionalidades implementadas

Este documento describe cada funcionalidad que ya existe en el prototipo `frontend/src/ModuloRRHH.jsx`. Está pensado para que un equipo nuevo pueda tomar el código y entender qué hace cada parte, o para reconstruirlo desde cero respetando el comportamiento.

Convenciones:
- ✅ = funcional
- 🔶 = funcional con limitaciones
- ❌ = no implementado (ver `05-pendientes.md`)

---

## 1. Estructura general de la aplicación

### 1.1 Vistas principales (tabs)
- ✅ **RRHH Control** — vista de la jefa de RRHH (presupuestos, KPIs, BD candidatos)
- ✅ **Pre-carga** — vista del reclutador para subir CVs
- ✅ **Base de datos CVs** — listado con filtros
- ✅ **Evaluador técnico** — vista del Ing. para calificar CVs
- ✅ **Mis KPIs (reclutador)** — métricas de desempeño individual

### 1.2 Simulación de tiempo
- ✅ Toggle "Simular CV ingresado" para pruebas sin subir PDF real
- ✅ Slider de "días simulados" para ver cómo evolucionan deadlines y standby

---

## 2. Gestión de Presupuestos (PRs)

### 2.1 Catálogo de PRs
- ✅ Listado con tarjetas: código, nombre, sede, estado, plazas solicitadas, evaluador asignado
- ✅ Estados: `sin_caracterizar` / `activo` / `pausado` / `terminado`
- ✅ Filtros visuales por estado
- ✅ Indicador "X% caracterizado" cuando le falta speech base

### 2.2 Crear nuevo PR
- ✅ Dialog modal con: código (auto-sugerido), nombre, sede, **evaluador técnico asignado**, fecha inicio, fecha fin
- ✅ Validaciones: campos obligatorios, fecha fin > fecha inicio, evaluador requerido
- ✅ Permite mismo código en sedes distintas (no valida unicidad por código)
- ✅ Estado inicial: `sin_caracterizar`

### 2.3 Detalle del PR
- ✅ Caja de evaluador técnico asignado (editable mediante select)
- ✅ Pills de estado (Activar / Pausar / Terminar) con validación de caracterización
- ✅ Sección **Parametrización de Categorías**:
  - Habilitar/deshabilitar Operarios / Oficiales / Ayudantes
  - Speech base por categoría (textarea)
  - Lista de cargos específicos derivados de los requerimientos abiertos
  - Speech personalizado opcional por cargo (con badge "✓ usa speech de [Categoría]" cuando hereda)
- ✅ Sección **Requerimientos**:
  - Crear req con nombre, deadline, comentario
  - Inputs por categoría (Operarios / Oficiales / Ayudantes)
  - **Botón "🔒 Fijar req"**: bloquea cantidades hasta usar "Modificar req"
  - **Botón "✏️ Modificar req"**: desbloquea temporalmente para editar
  - Crear cargos específicos al vuelo dentro del req (input + Enter o "+ Crear")
  - Validación de suma: cargos específicos deben sumar el total de la categoría
  - Botón "Cerrar incompleto" requiere comentario ≥5 chars
  - Botón "Eliminar" con confirmación

### 2.4 Reglas de caracterización
- ✅ Un PR está "caracterizado" si **todas las categorías solicitadas tienen speech base**
- ✅ Los cargos específicos NO requieren speech propio — heredan del speech base
- ✅ Solo PRs caracterizados pueden activarse
- ✅ PR sin requerimientos abiertos = no caracterizado

---

## 3. Pre-carga de CVs (Pipeline obreros — etapa 0)

### 3.1 Modo individual
- ✅ Drag-and-drop o selector de archivos (PDF principal + adjuntos)
- ✅ Validación: max 5 MB, solo PDF
- ✅ Marcar CV principal cuando hay múltiples archivos
- ✅ Detección de múltiples personas en mismo set (alerta)
- ✅ Llamada a IA para extraer:
  - Nombres, apellidos, DNI, fecha nacimiento, celular, email
  - Cargo principal, cargo secundario, otros cargos
  - Experiencia (rango), conocimientos
  - Estudios, nivel educativo
- ✅ Llamada a IA #2 para evaluación detallada:
  - Empresas con período, rol, sector metalmecánico, tamaño
  - Puntajes desglosados (años, sector, tamaño, progresión, estudios, CORMEI)
  - Justificación textual (3-4 líneas)
- ✅ Badge "IA ✓" en cada campo prellenado
- ✅ Validación de DNI (8 dígitos) + check duplicado en BD
- ✅ Foto del candidato — **solo upload manual** (extracción automática deshabilitada)

### 3.2 Modo masivo (bulk)
- ✅ Toggle "Carga masiva" — cada PDF se procesa como CV independiente
- ✅ Procesamiento en cola con loop `for...of` y functional updaters (sin loops infinitos)
- ✅ Lista visual con estado de cada CV: ⏳ en cola / ⚙ procesando / ✓ completo / procesado / ✗ falló
- ✅ Botón "✕" para eliminar CV individual del lote
- ✅ Indicador "⚠ N prob." con tooltip cuando hay campos faltantes o DNI duplicado
- ✅ Panel naranja inline detallando problemas del CV activo
- ✅ Navegación entre CVs con Anterior/Siguiente
- ✅ Sincronización form ↔ bulkCV en vivo (correcciones persisten al navegar)
- ✅ Persistencia del lote al guardar individual (volver al lote masivo)
- ✅ Auto-carga del primer CV procesado cuando termina la cola
- ✅ Manejo de archivos rechazados (>5MB, no-PDF) marcados como `failed`

### 3.3 Información detallada de cargo
- ✅ Tooltip "i" en cada campo muestra detalle de la IA
- ✅ Cargo principal: muestra "X año(s) en este cargo" calculado de empresas
- ✅ Cargo secundario: idem
- ✅ Experiencia: tabla de empresas con inicio/fin/empresa/años + total

### 3.4 Edición manual del disgregado de experiencia
- ✅ Cada fila de empresa tiene input editable de años (paso 0.1)
- ✅ Al editar:
  - Fila se marca con fondo amarillo + icono ✎
  - Recalcula total y nota Eval IA en vivo
  - Borra badge "IA ✓" del campo "Experiencia general" (marca cambio manual)
  - Cambia label "Total detectado" → "Total"
- ✅ Marca `_editadoManual: true` en evalu para auditoría

### 3.5 Validaciones para guardar
- ✅ Campos obligatorios: nombres, apellidos, DNI (8 dígitos), fecha nacimiento, celular, cargo principal, experiencia
- ✅ DNI no debe existir en BD
- ✅ Email opcional (se valida formato si está)
- ✅ Foto opcional

### 3.6 Pantalla post-guardado
- ✅ Resumen del candidato con Eval IA, Eval Técnica, etc.
- ✅ En modo bulk: botones "← Volver al lote masivo" + "Ir a base de datos →"
- ✅ En modo individual: "Cargar otro candidato" + "Ir a base de datos →"

---

## 4. Pipeline obreros (etapas 1-3)

### 4.1 Etapa 1: Filtro RRHH / Oferta
- ✅ Reclutador selecciona PR destino y categoría a ofrecer
- ✅ Genera speech automático heredado de la parametrización
- ✅ Captura de constancias:
  - WSP foto (de aceptación)
  - Datos del candidato (DNI, edad, etc.) confirmados
- ✅ Botón "Enviar para Evaluación Técnica →" con countdown 4s y opción de cancelar
- ✅ Tras enviar: auto-avanza a etapa 2 en 3s

### 4.2 Filtro técnico — reglas por categoría (helper `requiereEvalTecnica`)
- ✅ **Ayudante:** NUNCA pasa por filtro técnico
- ✅ **Oficial:** 20% aleatorio determinístico (`id % 5 === 0`)
- ✅ **Operario u otra:** SIEMPRE pasa
- ✅ Cuando NO requiere: panel verde con botón "Avanzar directo a contratación →" salta de etapa 1 a 3 marcando `evalTecCompletado:true, evalTecOmitida:true`

### 4.3 Etapa 2: Evaluación Técnica
- ✅ Vista para el Ing. evaluador (solo PRs adjudicados a él)
- ✅ Calificación 1-100 con razón opcional
- ✅ Si nota ≤29: descalificado
- ✅ Auto-avanza a etapa 3 en 3s
- ✅ Notas del evaluador con esquema de colores: ≥86 verde oscuro, ≥50 verde, ≥40 naranja, ≥30 naranja claro, <30 rojo

### 4.4 Etapa 1 vista post-eval-técnica (decisión del reclutador)
Pills de decisión:
- ✅ **A) Ofrecer categoría** (no disponible si ambos descalificaron)
- ✅ **B) Ofrecer otra categoría** (cambiar categoría dentro del mismo PR)
- ✅ **C) Standby** (pausa, registra timestamp para KPI)
- ✅ **D) 📤 Enviar a otro PR** (solo si ambosDescalif o tecMenor; requiere PR alternativo con otro evaluador)

### 4.5 Reasignación a otro PR
- ✅ Panel inline púrpura con:
  - Dropdown de PRs activos con OTRO evaluador
  - Dropdown de categoría destino
  - Razón obligatoria si ambos descalificaron (mín 10 chars)
- ✅ Al confirmar:
  - Cambia `prSeleccionado`
  - Reinicia eval téc
  - Reinicia etapa a 1
  - Genera nueva oferta con speech del PR destino
  - Limpia constancias previas
  - Agrega entrada al `interaccionesHistorial`

### 4.6 Etapa 3: Contratación
- ✅ Datos finales del candidato
- ✅ Confirmación de contratación
- ✅ Marca el req como avanzando hacia "cumplido"

---

## 5. Base de datos de candidatos

### 5.1 Listado
- ✅ Cards con foto/avatar, nombre, DNI, edad, cargo, evaluación IA con colores unificados
- ✅ Indicadores: días sin revisar (rojo si >X), evaluación técnica, etapa actual
- ✅ Filtros por etapa, por PR asignado, por estado standby
- ✅ Sección destacada "Personas para tener atención" (ambos descalificados pero contratados)

### 5.2 Modal de candidato
- ✅ Pestañas por etapa con estado (completada / actual / pendiente)
- ✅ Histórico de interacciones (timeline)
- ✅ CV expandible inline con datos extraídos
- ✅ Vista de evaluación IA con desglose por dimensiones
- ✅ Botones de acción según etapa actual

---

## 6. KPIs

### 6.1 KPIs del reclutador (Mis KPIs)
- ✅ **Tiempo promedio cierre vacante** (días desde solicitud a contratación)
- ✅ **Tasa de conversión por etapa** (% que pasa de etapa N a N+1)
- ✅ **Vacantes cerradas por período** (semana / mes)
- ✅ **Calidad post-contratación** (nota promedio del Ing. técnico para los CVs presentados)
- ✅ **Tasa de dropout** (% candidatos que abandonan o fueron rechazados)
- ✅ Comparativa con promedio del equipo (cuando aplica)

### 6.2 KPIs RRHH Control
- ✅ Plazas cubiertas vs solicitadas por PR
- ✅ Días desde apertura del req
- ✅ Estado de caracterización
- ✅ Casos para atención (bandera roja)

---

## 7. Evaluadores técnicos

### 7.1 Modelo
- ✅ Constante `EVALUADORES` con id, nombre, color, especialidad
- ✅ 3 evaluadores mock: Carlos Mendoza (soldadura), Rosa Salinas (estructuras), Jorge Lazarte (calidad)
- ✅ Cada PR tiene `evaluadorId` (uno solo)

### 7.2 Vista del evaluador
- ✅ Lista de CVs pendientes (filtra solo los de PRs adjudicados a él)
- ✅ Cada CV con datos básicos + Eval IA
- ✅ Input de nota 1-100 + razón
- ✅ Histórico de CVs ya evaluados con fecha y nota

### 7.3 KPIs del evaluador
- ✅ CVs revisados en últimos 30/180 días
- ✅ Tiempo promedio de revisión (de envío a calificación)
- ✅ Distribución de notas dadas

---

## 8. Modal de candidato — interacciones especiales

### 8.1 Standby
- ✅ Reclutador puede pausar al candidato
- ✅ Registra `sbBaseTs` y `sbStandby:true`
- ✅ Aparece badge en BD card

### 8.2 Cambio de categoría (Ofrecer otra)
- ✅ Genera nuevo speech con la nueva categoría
- ✅ Reinicia constancias para nueva negociación
- ✅ Si requiere razón (porque ambos descalificaron): textarea con mín 10 chars

### 8.3 Histórico de interacciones
- ✅ Cada decisión (ofrecer, rechazar, standby, cambio_categoria, envio_otro_pr, contratado) queda registrada con timestamp
- ✅ Tipos: `oferta_inicial`, `cambio_categoria`, `standby`, `envio_otro_pr`, `contratado`, `rechazado`
- ✅ Cada entrada incluye razón cuando aplica

---

## 9. UI/UX

### 9.1 Esquema de colores unificado (notas)
Aplicado consistentemente en CvPreview, post-save, modal candidato, BD card, evaluador técnico:
- ✅ ≥86 → #085041 (verde oscuro / "Sobresaliente")
- ✅ ≥50 → #0a6e3f (verde / "Operario")
- ✅ ≥40 → #b36200 (naranja / "Oficial")
- ✅ ≥30 → #BA7517 (naranja claro / "Ayudante")
- ✅ <30 → #b00020 (rojo / "Descalificado")

### 9.2 Paleta industrial
- ✅ Dark navy: #0C447C, #185FA5
- ✅ Orange: #EF9F27, #BA7517
- ✅ Steel blue: #5DCAA5, #7AB8F5
- ✅ Background neutro: #fafbfc, #f7f8fa, #fff

### 9.3 Componentes reutilizables
- ✅ `Section` — caja con título de sección
- ✅ `Row` — fila con label + badge IA + tooltip "i" + contenido
- ✅ `InfoBtn` / `InfoPanel` — botón "i" expandible con datos detallados
- ✅ `AiBadge` — pill verde "IA ✓"
- ✅ `EvalTecCard` — caja para nota técnica
- ✅ `CvPreview` — vista compacta de datos extraídos
- ✅ `ErrorBoundary` — captura errores y muestra UI de fallback

### 9.4 Manejo de errores
- ✅ ErrorBoundary class wrappea el componente principal
- ✅ Listener global `window.addEventListener("error")` para logging
- ✅ Botón "Reintentar" en pantalla de error con stack trace visible

---

## 10. Helpers globales

| Helper | Propósito |
|---|---|
| `calcEdad(fechaNac)` | Calcula edad desde fecha de nacimiento |
| `requiereEvalTecnica(c)` | Determina si un candidato pasa por filtro técnico |
| `evaluarCaracterizacion(p)` | Verifica si un PR tiene speeches completos |
| `extraerFotoCarnetDeDocs(...)` | (deshabilitado) Extracción de foto vía PDF.js + Vision |
| `generarAvatarFallback(nom, ape)` | Genera SVG con iniciales como fallback de foto |

---

## 11. Constantes y enumeraciones

| Constante | Contenido |
|---|---|
| `RECLUTADORES` | 4 mocks (Ana, Luis, María, Pedro) con color |
| `EVALUADORES` | 3 mocks (Carlos, Rosa, Jorge) con especialidad |
| `CAT_LABELS` | `{operario:"Operarios", oficial:"Oficiales", ayudante:"Ayudantes"}` |
| `CAT_COLORS` | Paleta por categoría (col, bg, bord) |
| `EXP_RANGOS` | 8 rangos: Sin experiencia, Menos de 1 año, 1–2 años, 3–5, 6–10, +10, +15, +20 |
| Motivos de rechazo | 7 tipificados |

---

## 12. Estado del prototipo (resumen)

- 7,600+ líneas en un solo archivo JSX
- Parser Babel JSX limpio (sin errores de sintaxis)
- Component principal `ModuloCargaCVInner` envuelto por `ModuloCargaCV` con ErrorBoundary
- Sin TypeScript (recomendado migrar antes de escalar)
- Sin tests automatizados
- Sin separación en archivos por dominio
