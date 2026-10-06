# CLAUDE.md — Analizador L5-S1

## Contexto

Trabajo Integrador de Biomecánica (FCEFyN-UNC, 2026), Grupo 21. Herramienta web que, a partir de una foto de perfil de un levantamiento de carga, calcula el momento en L5-S1, la fuerza de los erectores, la compresión lumbar y la clasifica según NIOSH. Se compara una postura en sentadilla (espalda recta) con una postura de flexión de espalda.

Los integrantes son estudiantes de Ingeniería Biomédica. Se defiende oralmente: **todo el grupo tiene que poder explicar cada cálculo**.

## Reglas para trabajar en este repo

- Responder y comentar el código **en español**.
- **Explicar cada cambio en lenguaje simple** antes o después de hacerlo, para que el grupo lo entienda y lo pueda defender. Preferir cambios chicos y comprensibles.
- **Las decisiones del modelo biomecánico las toma el grupo** (segmentos, tablas, ecuaciones, supuestos). Si un cambio altera el modelo, proponerlo y esperar confirmación; no cambiarlo por cuenta propia.
- Mantener la página como **un solo archivo `index.html` sin dependencias ni paso de compilación** (HTML + CSS + JS inline; solo Google Fonts como recurso externo).
- La foto nunca se envía a ningún servidor: todo se procesa en el navegador.
- Después de tocar cálculos, **verificar con el ejemplo incluido**: masa 78 kg, carga 15 kg, masculino, referencia 32 cm, erectores 5,9 cm → θ ≈ 60,0°, M ≈ 178,7 N·m, Fm ≈ 3029 N, Fc ≈ 3291 N. Si cambian, explicar por qué.

## Modelo actual (decidido por el grupo)

- Estático, 2D, plano sagital. g = 9,81 m/s².
- Segmentos sobre L5-S1 (de Leva 1996, % masa / % CM desde el extremo proximal):

| Segmento | Extremos | Masc. | Fem. |
|---|---|---|---|
| Cabeza y cuello | Vértex → C7 | 6,94 / 50,02 | 6,68 / 48,41 |
| Tronco sobre L5-S1 (UPT+MPT) | C7 → L5-S1 | 32,29 / 50,7 | 30,10 / 49,6 |
| Brazo (×2) | Hombro → Codo | 2,71 / 57,72 | 2,55 / 57,54 |
| Antebrazo (×2) | Codo → Muñeca | 1,62 / 45,74 | 1,38 / 45,59 |
| Mano (×2) | Muñeca → 3er metacarpiano | 0,61 / 79,00 | 0,56 / 74,74 |

  El % CM del tronco sobre L5-S1 es una estimación (combinación de tronco superior y medio); declarada como simplificación.
- M = Σ mᵢ·g·dᵢ (+ = flexor). Fm = M / 0,059 m (si M < 0, Fm = 0).
- θ = inclinación de C7→L5-S1 respecto de la vertical (+ = flexión hacia adelante).
- Fc = Fm + (P_sup + P_carga)·cos θ ; Fs = (P_sup + P_carga)·sin θ.
- NIOSH: < 3400 N verde; 3400–6400 N ámbar; > 6400 N rojo.
- Ángulo de rodilla ≤ 140° → "tipo sentadilla"; > 140° → "flexión de espalda" (orientativo).

## Estructura del código (`index.html`)

- `POINTS` / `HELP`: puntos a marcar y su ayuda.
- `DELEVA` / `SEGS`: parámetros antropométricos y segmentos.
- `compute()`: todo el modelo biomecánico (escala, momentos, fuerzas, ángulos).
- `draw()`: dibujo sobre el canvas.
- `update()`, `renderCmp()`, `exportText()`: interfaz, comparación A/B y texto para Excel.
- `loadDemo()`: escena de ejemplo con puntos precargados.

## Pendientes / ideas

- Completar la cita del brazo de palanca de 5,9 cm.
- Probar con fotos reales de perfil (postura A sentadilla, postura B flexión) y validar contra Tracker + Excel.
- Zoom en la foto para marcar con más precisión.
- Posible extensión dinámica: video cuadro a cuadro.
