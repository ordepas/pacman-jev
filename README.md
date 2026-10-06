# PAC-MAN x JEV — Solo Brain Edition

Pac-Man donde **JEV (TypeSafe System One) es el Único cerebro**. Sin AI local. JEV decide cada movimiento.

## Arquitectura

```
┌─────────────┐     220ms      ┌──────────────┐
│  Game State  │ ──────────────►│  JEV (API)   │
│  - Pac-Man   │    pregunta    │  System One  │
│  - Ghosts    │◄──────────────│  Model       │
│  - Dots      │   dirección    └──────────────┘
│  - 5x5 grid  │
└─────────────┘
        │
        ▼
  Pac-Man sigue dirección de JEV
  hasta nueva decisión (~2 ticks)
```

### Estado enviado a JEV (ultra-compacto)
```
P(10,16) GHOSTS:[Rd5 Ld8 sUd12] DOTS:[(8,14)3 (12,16)5] 
GO:[up,down,left] GRID: #####|# P #|#...#|#   #|##### 
Lv1 Sc290 Left180
```

- **P(x,y)**: posición Pac-Man
- **GHOSTS**: dirección + distancia de cada fantasma (`s` = scared)
- **DOTS**: 3 dots más cercanos con distancia
- **GO**: direcciones disponibles
- **GRID**: vecindario 5x5 (P=pacman, #=pared, .=dot, O=power pellet, g=fantasma)

### Criterios dinámicos
JEV recibe info por cada dirección disponible:
```
up→(10,15) exits:3 ghost:5 dot
down→(10,17) exits:2 ghost:8
left→(9,16) exits:3 ghost:12
```

## Resultados de prueba (JEV Solo Brain)

| Métrica | Valor |
|---------|-------|
| Score | 1,250 |
| Supervivencia | 40 segundos |
| Dots recolectados | 81/205 (40%) |
| Nivel alcanzado | 1 |
| JEV Calls | 143 |
| Latencia promedio | 229ms (min:199ms, max:357ms) |
| Confianza promedio | ~20% |
| Score/min | 1,875 |
| Dots/min | 122 |

### Patrones observados
- **Primer vida**: ~30 segundos (el más largo)
- **Vidas posteriores**: mueren en ~5 segundos (reaparecer = vulnerable)
- **Confianza alta (50-95%)** en corredores abiertos con dots claros
- **Confianza baja (3-20%)** en intersecciones complejas
- **Oscilación**: JEV tiende a alternar UP/DOWN cuando no está seguro

### Limitaciones conocidas
1. **Latenia ~220ms** = reacciona 2 ticks tarde a movimientos de fantasmas
2. **Sin memoria** — JEV no recuerda decisiones pasadas ni vidas anteriores
3. **Visibilidad limitada** — solo ve vecindario 5x5 + posiciones de fantasmas
4. **Spawn kill** — fantasmas cerca del punto de reaparición matan rápido

## Comparación con versión AI local + JEV overlay (branch `main`)

| Aspecto | AI Local + JEV (`main`) | JEV Solo Brain (`jev-only`) |
|---------|------------------------|---------------------------|
| Decisiones | AI local tácticas + JEV estratégico | Solo JEV |
| Latencia efectiva | 0ms (local) + 220ms (JEV) | ~220ms cada decisión |
| Supervivencia | 5+ minutos | ~40 segundos |
| Nivel alcanzado | 3-4 | 1 |
| Score típico | 8,000-12,000 | ~1,250 |

## Instalación

```bash
git checkout jev-only
# Abrir http://localhost:4000 (requiere server.js con proxy a TypeSafe API)
```

## API Key

TypeSafe API key está hardcodeada en `index.html` para demo. En producción, usar variables de entorno.

## Servidor

```bash
node server.js  # Puerto 4000, proxy CORS para TypeSafe API
```

Servicio systemd:
```bash
sudo systemctl start pacman-jev
```