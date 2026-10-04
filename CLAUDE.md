# AudiBIM — Clase BIM + IA

> Documento vivo del proyecto. El método de trabajo general está en `~/.claude/CLAUDE.md`. Aquí va solo el contexto de este proyecto.

## Qué es
Repo de la clase **BIM + IA** (comunidad ECD — Escuela de Capacitación Digital).
Clonado desde: https://github.com/ComunidadECD/Repo01-AudiBIM (remoto `origin`, rama `main`).
Carpeta de la clase: `Documents\bim-ia\` (aquí irán los siguientes repos: Repo02, Repo03…).

**AudiBIM** es un add-in de Revit 2027 en C# (WPF, `net10.0-windows`). Audita la calidad de un modelo BIM con reglas escritas en Markdown (`.md`): categoría + parámetro + operador + severidad + color. Resalta, aísla o selecciona los incumplimientos en Revit y exporta un informe PDF/HTML.

## Arquitectura (resumen)
- `App.cs` → registra el Ribbon (pestaña *Automatización BIM* → panel *Auditoria* → botón *AudiBIM*).
- `Commands/OpenAuditorCommand.cs` → abre la ventana WPF (no modal).
- `Revit/ExternalEventController.cs` → puente seguro entre WPF y la API de Revit (IExternalEventHandler).
- `Parsing/RuleParser.cs` → lee las reglas `.md`.
- `Services/RuleEngine.cs` → evaluación determinista de las reglas. `AuditService.cs` coordina.
- `Services/*` → recolección de elementos, parámetros, overrides gráficos, aislamiento, PDF y persistencia en AppData.
- `UI/` → MVVM (MainViewModel, RuleTabViewModel, MainWindow, tooltip flotante).

## Entorno (PC de Larry)
- Revit 2027 instalado en `C:\Program Files\Autodesk\Revit 2027\` (las referencias del `.csproj` resuelven).
- .NET SDK 10 → instalado vía winget el 2026-10-04.
- Compilar: `dotnet build BIMQualityAuditor.csproj`. **Revit debe estar cerrado** porque bloquea la DLL.
- El `.addin` del repo apunta a la ruta del profesor (`d:\Antigravity\...`). **No lo editamos en el repo**: se instala una copia con la ruta local en `%APPDATA%\Autodesk\Revit\Addins\2027\`.

## Bitácora
### 2026-10-04
- Se creó `Documents\bim-ia\` y se clonó el repo (1 commit: "Initial commit").
- Se instaló el .NET SDK 10.
- Compilado en Release (`dotnet build -c Release`) → `bin\Release\net10.0-windows\BIMQualityAuditor.dll`. 0 errores, 0 advertencias.
- **Bug del repo corregido:** `App.cs` carga `AudiBIM_32.png` y `AudiBIM_16.png` para el ícono del Ribbon, pero el `.csproj` no los copiaba al output, así que el botón salía sin ícono. Se agregaron como `Content` en el `.csproj` Commit `3981851` en la rama `fix/iconos-ribbon`, rebasada sobre el main 2026 del profesor y subida al fork para PR a ComunidadECD. Candidato a reportar al profesor.
- Manifiesto instalado en `%APPDATA%\Autodesk\Revit\Addins\2027\AudiBIM.addin` apuntando a la DLL de Release (la carpeta `2027` no existía; se creó).
- Para que Revit cargue el add-in hay que reiniciar Revit. Después de eso, para recompilar hay que cerrar Revit primero (bloquea la DLL).
- El repo **no trae modelo Revit**: no hay `.rvt` ni LFS, y tampoco releases ni otras ramas. Para probar se usa el sample de Autodesk, copiado a `bim-ia\modelos\Snowdon Towers Sample Architectural.rvt` (sin atributo de solo lectura).
- Reglas de prueba en `bim-ia\reglas-prueba\reglas_prueba.md` (R01 muros/Comentarios, R02 puertas/Marca, R03 muros/Nivel).
- **Gotcha del parser:** cualquier línea que empiece con `# Regla…` (por ejemplo, un título "# Reglas de prueba") abre un bloque nuevo. No poner títulos así en los `.md` de reglas.
- **Gotcha de nombres:** el PARAMETRO se busca por nombre visible, que depende del idioma de Revit (`Comentarios` vs `Comments`). Excepciones: `Nivel`/`Level` y `Tipo`/`Type Name`. La CATEGORIA acepta inglés o español. Solo se auditan elementos visibles en la vista activa.
- Se creó este `CLAUDE.md` y se versiona en la rama `larry`.

## Git (esquema de trabajo)
- Cuenta de GitHub para este trabajo: **lsalasg** (creada 2026-10-04). Identidad configurada **solo en este repo**: `lsalasg <337861139+lsalasg@users.noreply.github.com>` (correo privado de GitHub). El git global sigue con la identidad personal.
- Remotos: `upstream` = repo del profesor (ComunidadECD) · `origin` = fork `lsalasg/Repo01-AudiBIM`.
- Ramas: `main` = copia limpia del profesor (no se toca) · `larry` = rama de trabajo del día a día · ramas de tema (`fix/…`, `feature/…`) salen de `main` solo para PR al profesor.
- Traer cambios del profesor: `git switch main` → `git pull upstream main` → `git switch larry` → `git merge main`.
- **Decisión 2026-10-04: Larry se queda en Revit 2027.** El profesor migró el repo a Revit 2026 (.NET 8) en `979ba73` y `0e1124c`. Al hacer merge de `main` a `larry` se mantiene Revit 2027 / `net10.0-windows` en el `.csproj`, el `.addin` y la etiqueta de `MainWindow.xaml`. En cada merge futuro del profesor hay que revisar que no vuelva a 2026. En esta PC no hay Revit 2026.
- Objetivo: figurar como colaborador del profesor (`angellofinetti` / ComunidadECD) y que vea los commits. Colaborador = lo agrega él en Settings → Collaborators (usuario `lsalasg`). Los commits los ve vía PR desde `lsalasg:fix/iconos-ribbon` → `ComunidadECD:main`.
