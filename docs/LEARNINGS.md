# Habilidades y Proceso de Trabajo

## 1. Organización de Librerías KiCad

### Problema
Múltiples carpetas con archivos duplicados (.kicad_mod, .kicad_sym, .step).

### Proceso
1. **Escanear directorio**: Buscar todos los archivos por extensión
2. **Identificar duplicados**: Comparar nombres de archivo
3. **Verificar contenido**: Usar `md5` para detectar archivos idénticos
4. **Crear estructura limpia**:
   - `symbols.kicad_sym` (un solo archivo con todos los símbolos)
   - `footprints.pretty/` (carpeta con footprints individuales)
   - `3dmodels/` (carpeta con modelos STEP)
5. **Mergear símbolos**: Script Python para combinar .kicad_sym únicos

---

## 2. Extracción de Footprints desde PCB

### Proceso
```bash
# 1. Extraer nombres de footprints del PCB
grep -o '(footprint "[^"]*"' archivo.kicad_pcb | sed 's/(footprint "//;s/"//' | sort -u

# 2. Verificar cuáles faltan en la librería
while read fp; do
  name=$(echo "$fp" | sed 's/.*://')
  if [ -f "footprints.pretty/$name.kicad_mod" ]; then
    echo "✅ $fp"
  else
    echo "❌ $fp (FALTA)"
  fi
done < footprints_faltantes.txt

# 3. Buscar en librerías estándar de KiCad
find /Applications/KiCad/KiCad.app/Contents/SharedSupport/footprints -name "NombreFootprint.kicad_mod"

# 4. Copiar a la librería
cp /ruta/al/footprint.kicad_mod footprints.pretty/
```

---

## 3. Licencias

### Ubicación de licencias en macOS
```bash
# KiCad bundle
ls /Applications/KiCad/KiCad.app/Contents/Resources/Licenses/

# Solo contiene Python/LICENSE.txt (PSF License)
# NO hay licencia KiCad como archivo separado
```

### KiCad usa GPL v2+
- El binario de KiCad contiene la licencia GPL v2+ embebida
- Los proyectos y librerías creados con KiCad deben ser compatibles
- **Solución**: Crear archivo `LICENSE` manualmente con texto GPL v2+

### Crear LICENSE
```bash
# Descargar texto oficial
curl -o LICENSE https://www.gnu.org/licenses/old-licenses/gpl-2.0.txt

# O crear manualmente con el texto de gnu.org
```

---

## 4. Git Push con Token (HTTPS)

### Configuración
```bash
# Verificar usuario git local
git config user.name
git config user.email

# Verificar remote
git remote -v
```

### Proceso de Push
```bash
# 1. Temporalmente configurar remote con token
git remote set-url origin https://USUARIO:TOKEN@github.com/USUARIO/REPOSITORIO.git

# 2. Push
git push -u origin main

# 3. IMPORTANTE: Limpiar remote (quitar token)
git remote set-url origin https://github.com/USUARIO/REPOSITORIO.git
```

### Seguridad
- **NUNCA** commitear tokens en el repositorio
- **NUNCA** mostrar tokens en logs o pantallas
- **SIEMPRE** limpiar la URL después del push
- El token se almacena en la config global de git (no en el repositorio)

---

## 5. Verificación de Contenido

### Contar archivos
```bash
# Footprints
ls footprints.pretty/*.kicad_mod | wc -l

# Símbolos (entradas principales)
grep -c '(symbol "' symbols.kicad_sym

# Modelos 3D
ls 3dmodels/*.step | wc -l
```

### Detectar duplicados
```bash
# Por nombre
find . -name "*.kicad_mod" -exec basename {} \; | sort | uniq -c | sort -rn

# Por contenido (hash MD5)
find . -name "*.kicad_mod" -exec md5 -r {} \; | awk '{print $1}' | sort | uniq -d
```

---

## 6. Estructura de Repositorio KiCad

```
repositorio/
├── symbols.kicad_sym      # Todos los símbolos en un archivo
├── footprints.pretty/      # Carpeta con footprints (.kicad_mod)
├── 3dmodels/              # Modelos STEP (.step)
├── LICENSE                # GPL v2+ (obligatorio)
├── README.md              # Documentación
└── .gitignore             # Excluir: *.zip, *.bak, .DS_Store, etc.
```

---

## 7. Script de Mergear Símbolos

```python
#!/usr/bin/env python3
import os
import re

def merge_symbols(source_dirs, output_file):
    """Combina múltiples .kicad_sym en uno solo"""
    unique_symbols = {}
    
    for source_dir in source_dirs:
        for root, dirs, files in os.walk(source_dir):
            for filename in files:
                if filename.endswith('.kicad_sym'):
                    filepath = os.path.join(root, filename)
                    with open(filepath, 'r') as f:
                        content = f.read()
                        symbols = re.findall(
                            r'\(symbol\s+"[^"]+".*?\n\s+\)\s*\)',
                            content, re.DOTALL
                        )
                        for symbol in symbols:
                            name = re.search(r'\(symbol\s+"([^"]+)"', symbol)
                            if name and name.group(1) not in unique_symbols:
                                unique_symbols[name.group(1)] = symbol
    
    with open(output_file, 'w') as f:
        f.write('(kicad_symbol_lib (version 20211014) (generator custom)\n')
        for name, symbol in sorted(unique_symbols.items()):
            f.write('  ' + symbol + '\n')
        f.write(')\n')
    
    return len(unique_symbols)
```

---

## 8. Comandos Útiles de KiCad

```bash
# Buscar footprints en KiCad
find /Applications/KiCad -name "*.pretty" -type d

# Buscar un footprint específico
find /Applications/KiCad -name "Nombre.kicad_mod" -type f

# Verificar estructura de archivo .kicad_sym
head -20 archivo.kicad_sym
```

---

## 9. Gitignore para KiCad

```gitignore
# Archivos KiCad
*.kicad_pcb-bak
*.kicad_sch-bak
fp-info-cache

# ZIP (no subir)
*.zip

# Temporales
*.tmp
*.bak
*~

# macOS
.DS_Store

# Python
__pycache__/
*.pyc
```

---

## 10. Flujo de Trabajo Completo

1. **Explorar**: Encontrar todos los archivos en el directorio
2. **Deduplicar**: Identificar y eliminar duplicados
3. **Organizar**: Crear estructura limpia (symbols, footprints, 3d)
4. **Mergear**: Combinar símbolos en un solo archivo
5. **Verificar**: Contar archivos, detectar faltantes
6. **Documentar**: Actualizar README con contenido real
7. **Licenciar**: Agregar GPL v2+ (requerido por KiCad)
8. **Git init**: Inicializar repositorio
9. **Commit**: Guardar cambios
10. **Push**: Subir a GitHub (con token temporal)
11. **Limpiar**: Remover token del remote

---

## 11. Edge.Cuts con Esquinas Redondeadas en KiCad PCB

### Concepto
El contorno de la placa (Edge.Cuts) se define con **4 líneas rectas** (`gr_line`) y **4 arcos** (`gr_arc`) en las esquinas. Los arcos conectan los extremos de las líneas formando una placa con esquinas redondeadas.

### Formato correcto de `gr_arc`

```
(gr_arc
    (start X1 Y1)      ← PRIMER endpoint del arco (NO el centro)
    (mid Xm Ym)        ← punto medio sobre el arco
    (end X2 Y2)        ← SEGUNDO endpoint del arco
    (stroke
        (width 0.05)
        (type default)
    )
    (layer "Edge.Cuts")
    (uuid "...")
)
```

**CRÍTICO**: `(start)` es el **primer endpoint** del arco, NO el centro del círculo. `(end)` es el segundo endpoint. `(mid)` es un punto en el arco entre los dos endpoints.

### Error común (AI generando mal)
```
(gr_arc
    (start CX CY)      ← ¡MAL! Esto es el centro, no un endpoint
    (mid Xm Ym)
    (end X2 Y2)
)
```
KiCad no interpreta `(start)` como centro — lo interpreta como endpoint. Si se pone el centro, el arco se dibuja con ángulos incorrectos.

### Cómo calcular los puntos

Para una placa de **W × H mm** con esquinas de radio **R mm**:

```
     (X1,Y0)---- Línea Top ----(X2,Y0)
     /                                    \
  Arc TL                               Arc TR
   /                                      \
(X0,Y1)                              (X3,Y1)
  |                                          |
Lín.Izq                              Lín.Der
  |                                          |
(X0,Y2)                              (X3,Y2)
   \                                      /
  Arc BL                               Arc BR
     \                                    /
     (X1,Y3)---- Línea Bot ----(X2,Y3)
```

Donde:
- `X0 = left_x` (borde izquierdo)
- `X3 = right_x` (borde derecho)
- `Y0 = top_y` (borde superior)
- `Y3 = bottom_y` (borde inferior)
- `X1 = X0 + R`, `X2 = X3 - R` (empiezan las líneas rectas)
- `Y1 = Y0 + R`, `Y2 = Y3 - R` (empiezan las líneas rectas)

### Coordenadas de ejemplo (placa 99×99mm, radio 5mm)

```
Board: left=50.8, top=50.8, right=149.8, bottom=149.8
Radio: R = 5mm

X0=50.8  X1=55.8  X2=144.8  X3=149.8
Y0=50.8  Y1=55.8  Y2=144.8  Y3=149.8
```

### Las 4 líneas rectas

```
gr_line  Top:    start(X1,Y0) end(X2,Y0)     → (55.8,50.8)  → (144.8,50.8)
gr_line  Right:  start(X3,Y1) end(X3,Y2)     → (149.8,55.8) → (149.8,144.8)
gr_line  Bottom: start(X2,Y3) end(X1,Y3)     → (144.8,149.8)→ (55.8,149.8)
gr_line  Left:   start(X0,Y2) end(X0,Y1)     → (50.8,144.8) → (50.8,55.8)
```

### Los 4 arcos (ESQUINAS)

Fórmula del punto medio:
```
mid_x = center_x + R * cos(ángulo_medio)
mid_y = center_y + R * sin(ángulo_medio)
```
Donde `cos(45°) = sin(45°) ≈ 0.70711`, entonces `R × 0.70711 ≈ 3.536`

```
Corner    Center        start (1er endpoint)    mid (punto medio)         end (2do endpoint)
─────────────────────────────────────────────────────────────────────────────────────────────
Top-L     (X1,Y1)       (X0, Y1)                (X1-3.536, Y1-3.536)     (X1, Y0)
Top-R     (X2,Y1)       (X2, Y0)                (X2+3.536, Y1-3.536)     (X3, Y1)
Bot-R     (X2,Y2)       (X3, Y2)                (X2+3.536, Y2+3.536)     (X2, Y3)
Bot-L     (X1,Y2)       (X1, Y3)                (X1-3.536, Y2+3.536)     (X0, Y2)
```

### Valores concretos (99×99mm, R=5mm)

```
Top-Left Arc:
  start = (50.8, 55.8)          ← endpoint en línea izquierda
  mid   = (52.264, 52.264)      ← punto medio del arco
  end   = (55.8, 50.8)          ← endpoint en línea superior

Top-Right Arc:
  start = (144.8, 50.8)         ← endpoint en línea superior
  mid   = (148.336, 52.264)     ← punto medio del arco
  end   = (149.8, 55.8)         ← endpoint en línea derecha

Bottom-Right Arc:
  start = (149.8, 144.8)        ← endpoint en línea derecha
  mid   = (148.336, 148.336)    ← punto medio del arco
  end   = (144.8, 149.8)        ← endpoint en línea inferior

Bottom-Left Arc:
  start = (55.8, 149.8)         ← endpoint en línea inferior
  mid   = (52.264, 148.336)     ← punto medio del arco
  end   = (50.8, 144.8)         ← endpoint en línea izquierda
```

### Verificación: los endpoints deben coincidir

Cada endpoint de un arco debe coincidir exactamente con el extremo de una línea recta:
```
Top-Left Arc end   (55.8, 50.8)  = Top Line start   (55.8, 50.8)  ✓
Top-Right Arc end  (149.8, 55.8) = Right Line start  (149.8, 55.8) ✓
Bottom-Right Arc end (144.8,149.8)= Bottom Line start (144.8,149.8)✓
Bottom-Left Arc end  (50.8, 144.8)= Left Line start   (50.8, 144.8)✓
```

### Script Python para generar Edge.Cuts redondeados

```python
def generate_rounded_edge_cuts(width_mm, height_mm, radius_mm, offset_x=0, offset_y=0):
    """Genera Edge.Cuts con esquinas redondeadas para KiCad PCB"""
    import math
    
    x0 = offset_x
    y0 = offset_y
    x3 = offset_x + width_mm
    y3 = offset_y + height_mm
    x1 = x0 + radius_mm
    x2 = x3 - radius_mm
    y1 = y0 + radius_mm
    y2 = y3 - radius_mm
    
    mid_offset = radius_mm * math.cos(math.radians(45))  # ≈ 3.536 para R=5
    
    lines = []
    
    # 4 líneas rectas
    lines.append(f'\t(gr_line\n\t\t(start {x1} {y0})\n\t\t(end {x2} {y0})')
    lines.append(f'\t(gr_line\n\t\t(start {x3} {y1})\n\t\t(end {x3} {y2})')
    lines.append(f'\t(gr_line\n\t\t(start {x2} {y3})\n\t\t(end {x1} {y3})')
    lines.append(f'\t(gr_line\n\t\t(start {x0} {y2})\n\t\t(end {x0} {y1})')
    
    # 4 arcos (start=1er endpoint, end=2do endpoint, mid=punto medio)
    arcs = [
        (f'{x0}', f'{y1}', f'{x1-mid_offset:.3f}', f'{y1-mid_offset:.3f}', f'{x1}', f'{y0}'),  # Top-Left
        (f'{x2}', f'{y0}', f'{x2+mid_offset:.3f}', f'{y1-mid_offset:.3f}', f'{x3}', f'{y1}'),  # Top-Right
        (f'{x3}', f'{y2}', f'{x2+mid_offset:.3f}', f'{y2+mid_offset:.3f}', f'{x2}', f'{y3}'),  # Bot-Right
        (f'{x1}', f'{y3}', f'{x1-mid_offset:.3f}', f'{y2+mid_offset:.3f}', f'{x0}', f'{y2}'),  # Bot-Left
    ]
    
    for sx, sy, mx, my, ex, ey in arcs:
        lines.append(f'\t(gr_arc\n\t\t(start {sx} {sy})\n\t\t(mid {mx} {my})\n\t\t(end {ex} {ey})')
    
    return '\n'.join(lines)
```

### Checklist antes de guardar
1. ✅ `(start)` = primer endpoint (NO el centro)
2. ✅ `(end)` = segundo endpoint
3. ✅ `(mid)` = punto medio exacto (usar `R × 0.70711`)
4. ✅ Cada endpoint coincide con el extremo de una línea recta
5. ✅ Todos los elementos en `(layer "Edge.Cuts")`
6. ✅ Cada elemento tiene `(uuid "...")` único
7. ✅ Verificar con `grep "Edge.Cuts" archivo.kicad_pcb` — debe haber 8 elementos (4 líneas + 4 arcos)
