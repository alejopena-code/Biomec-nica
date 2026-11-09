# CLAUDE.md — Analizador L5-S1

## Contexto

Trabajo Integrador de Biomecánica (FCEFyN-UNC, 2026), Grupo 21. Herramienta web que, a partir de una foto de perfil de un levantamiento de carga, calcula el momento en L5-S1, la fuerza de los erectores, la compresión lumbar y la clasifica según NIOSH. Se compara una postura en sentadilla (espalda recta) con una postura de flexión de espalda.

Los integrantes son estudiantes de Ingeniería Biomédica. Se defiende oralmente: **todo el grupo tiene que poder explicar cada cálculo**.

## Reglas para trabajar en este repo

- Responder y comentar el código **en español**.
- **Explicar cada cambio en lenguaje simple** antes o después de hacerlo, para que el grupo lo entienda y lo pueda defender. Preferir cambios chicos y comprensibles.
- **Las decisiones del modelo biomecánico las toma el grupo** (segmentos, tablas, ecuaciones, supuestos). Si un cambio altera el modelo, proponerlo y esperar confirmación; no cambiarlo por cuenta propia.
- Mantener la página como **un solo archivo `index.html` sin paso de compilación** (HTML + CSS + JS inline). Recursos externos permitidos: Google Fonts y **MediaPipe** (detección de pose: librería desde cdn.jsdelivr.net y modelo desde storage.googleapis.com; decidido por el grupo el 2026-11-07). Cualquier otra dependencia, consultar antes.
- La foto, el video y la imagen de la cámara nunca se envían a ningún servidor: todo se procesa en el navegador.
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
- **La carga solo se suma si la persona la sostiene** (decisión del grupo, 2026-11-09). Foto: casilla "Sostiene la carga". Video: intervalo de agarre marcado a mano ("Agarra la carga aquí" / "Suelta la carga aquí"); sin intervalo = la sostiene todo el video. Fuera del intervalo, peso y momento de la carga = 0. Los cuadros de calibración tienen que estar dentro del intervalo.
- M = Σ mᵢ·g·dᵢ (+ = flexor). Fm = M / 0,059 m (si M < 0, Fm = 0).
- θ = inclinación de C7→L5-S1 respecto de la vertical (+ = flexión hacia adelante).
- Fc = Fm + (P_sup + P_carga)·cos θ ; Fs = (P_sup + P_carga)·sin θ.
- NIOSH: < 3400 N verde; 3400–6400 N ámbar; > 6400 N rojo.
- Ángulo de rodilla ≤ 140° → "tipo sentadilla"; > 140° → "flexión de espalda" (orientativo).

## Estructura del código (`index.html`)

- `POINTS` / `HELP`: puntos a marcar y su ayuda.
- `DELEVA` / `SEGS`: parámetros antropométricos y segmentos.
- `calcModel(P, inp)`: todo el modelo biomecánico (escala, momentos, fuerzas, ángulos). Función pura: recibe los puntos y los datos, no toca la página. `compute()` la llama con los puntos y el formulario actuales.
- Video: `loadVideo()`, `goFrame()`, `frames` (puntos por cuadro), `renderVideo()` (gráfico de Fc vs tiempo).
- Detección automática: `loadDetector()`, `detectFrame()`, `calibOf()` / `meanCal()` / `autoPoints()` (calibración opción B), `renderAutoRes()` (validación). `calSet` = cuadros de calibración, `autoSet` = cuadros automáticos.
- `draw()`: dibujo sobre el canvas.
- `update()`, `renderCmp()`, `exportText()`: interfaz, comparación A/B y texto para Excel.
- `loadDemo()`: escena de ejemplo con puntos precargados.

## Hoja de ruta (objetivo final: cámara en vivo)

1. ✅ Separar el cálculo (`calcModel`) de la interfaz.
2. ✅ Video cuadro a cuadro con marcado manual (cuasi-estático) → curva de Fc en el tiempo.
3. 🔧 Detección automática de pose (MediaPipe Pose Landmarker 0.10.14, modelo "full") sobre videos. **Decisión del grupo (2026-11-07): opción B, calibración con el propio sujeto.**
   - Detectados directamente: hombro, codo, muñeca, mano (punto medio índice–meñique ≈ 3er metacarpiano), cadera, rodilla, tobillo. Se usa el lado más visible.
   - Estimados con calibración: L5-S1 (respecto de cadera→hombro), C7 (hombro→oreja), vértex (oreja→nariz), carga (desplazamiento fijo desde la mano, ÷ largo cadera→hombro).
   - Cada punto estimado = origen + a·segmento + b·segmento girado 90°; (a, b) = promedio de los cuadros de calibración (marcados a mano).
   - Validación: validación cruzada en los cuadros de calibración (error en cm por punto y diferencia de Fc a mano vs automático).
   - Pendiente: probar con videos reales y comparar contra Tracker.
4. Cámara en vivo (suavizado, indicador NIOSH en tiempo real, registro del pico).

## Pendientes / ideas

- Completar la cita del brazo de palanca de 5,9 cm.
- Probar con fotos reales de perfil (postura A sentadilla, postura B flexión) y validar contra Tracker + Excel.
- Zoom en la foto para marcar con más precisión.
- Posible extensión dinámica (dinámica inversa con aceleraciones): requiere decisión del grupo.
