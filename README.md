# Analizador L5-S1

Herramienta web para estimar la carga mecánica sobre la articulación lumbosacra (L5-S1) durante el levantamiento manual de cargas, a partir de una foto de perfil.

**Trabajo Integrador — Biomecánica 2026, FCEFyN-UNC — Grupo 21**

## Qué hace

1. Se carga una foto de perfil del sujeto levantando una carga.
2. Se calibra la escala marcando los extremos de un objeto de largo conocido (termo de 32 cm).
3. Se marcan los puntos anatómicos: vértex, C7, L5-S1, hombro, codo, muñeca, mano (3er metacarpiano), centro de la carga y, opcionalmente, cadera, rodilla y tobillo.
4. La página calcula en vivo:
   - la inclinación del tronco (θ) y el ángulo de rodilla;
   - los brazos de palanca del cuerpo y de la carga;
   - el momento en L5-S1, la fuerza de los erectores, la compresión y la fuerza de corte;
   - la clasificación de riesgo según NIOSH (3400 N / 6400 N).
5. Permite guardar dos posturas (A y B), compararlas y copiar los resultados para pegarlos en Excel.
6. **Video cuadro a cuadro**: se carga un video, se marcan los puntos en un cuadro y se avanza con ▶ (o la flecha →). Los puntos se copian al cuadro siguiente para solo ajustar los que se movieron. Se obtiene el gráfico de la compresión a lo largo del tiempo, el pico y la tabla de todos los cuadros para Excel. Cada cuadro se calcula como una foto (análisis cuasi-estático, sin aceleraciones). Los cuadros por segundo del video se ingresan a mano (el navegador no los informa).

La foto y el video se procesan solo en el navegador; no se suben a ningún servidor.

7. **Detección automática (MediaPipe)**: se marcan a mano 3 o 4 cuadros de calibración y la página detecta la pose en el resto del video. L5-S1, C7, vértex y la carga (que MediaPipe no detecta) se ubican con la relación medida en los cuadros de calibración del propio sujeto. La página muestra la validación: error en cm de cada punto y diferencia de compresión entre lo automático y lo marcado a mano. Requiere internet la primera vez para descargar el detector (≈ 20 MB); el video igual se procesa solo en la computadora.

8. **La carga solo se suma cuando la persona la sostiene.** En una foto, con la casilla "Sostiene la carga". En un video, marcando el intervalo de agarre con "Agarra la carga aquí" y "Suelta la carga aquí" (se ve como una franja en el gráfico). Fuera de ese intervalo, el peso y el momento de la carga valen 0 (por ejemplo, cuando la persona se agacha a acomodar la caja sin levantarla).

9. **Cámara en vivo (en desarrollo):** con "Usar la cámara" se calibra con 3 o 4 capturas (cuenta regresiva, se congela la imagen y se marcan los puntos a mano) y después se ve en tiempo real el esqueleto, la compresión con su color NIOSH y el gráfico de los últimos 10 s. Por ahora se considera que la persona sostiene la carga todo el tiempo.

**Hoja de ruta:** el objetivo final es usar el modelo con la cámara en vivo (ver `CLAUDE.md`).

## Cómo usarla

No necesita instalación ni compilación. Abrir `index.html` con doble clic en cualquier navegador.

Para servirla localmente (opcional):

```bash
python -m http.server 8000
# abrir http://localhost:8000
```

## Modelo biomecánico

Modelo estático 2D en el plano sagital.

| Magnitud | Ecuación |
|---|---|
| Momento en L5-S1 | M = Σ mᵢ · g · dᵢ (dᵢ = distancia horizontal del CM del segmento a L5-S1) |
| Fuerza de los erectores | Fm = M / d_erectores (d_erectores = 5,9 cm) |
| Compresión | Fc = Fm + (P_sup + P_carga) · cos θ |
| Corte | Fs = (P_sup + P_carga) · sin θ |

θ = inclinación de la recta C7 → L5-S1 respecto de la vertical.

Parámetros segmentales: de Leva (1996). Se consideran solo los segmentos ubicados por encima de L5-S1: cabeza y cuello, tronco superior + medio, brazos, antebrazos y manos (×2 por simetría).

### Hipótesis y simplificaciones

- Segmentos rígidos, con masa y posición del CM fijas.
- Análisis estático: no se consideran aceleraciones.
- Todo el movimiento ocurre en el plano de la foto; ambos lados del cuerpo son simétricos.
- El momento externo lo equilibran solo los erectores espinales (sin cocontracción ni presión intraabdominal).
- El CM del "tronco sobre L5-S1" se estimó combinando los subsegmentos superior y medio de de Leva (~50 % de C7→L5-S1).
- L5-S1 se ubica en la foto de forma aproximada, a la altura de las crestas ilíacas.
- Clasificación del levantamiento: ángulo de rodilla ≤ 140° → tipo sentadilla; > 140° → flexión de espalda (orientativo).

## Estructura

```
index.html   Página completa (HTML + CSS + JS en un solo archivo, sin dependencias)
README.md    Este archivo
CLAUDE.md    Contexto del proyecto para asistentes de IA
```

## Referencias

- de Leva, P. (1996). Adjustments to Zatsiorsky-Seluyanov's segment inertia parameters. *Journal of Biomechanics*, 29(9), 1223–1230.
- NIOSH (1981). *Work practices guide for manual lifting* (DHHS (NIOSH) Publication No. 81-122). Origen de los límites 3400 N y 6400 N.
- Waters, T. R., Putz-Anderson, V., Garg, A., & Fine, L. J. (1993). Revised NIOSH equation for the design and evaluation of manual lifting tasks. *Ergonomics*, 36(7), 749–776.
- Brazo de palanca de los erectores (5,9 cm): *completar con la cita del trabajo utilizado.*

## Uso de inteligencia artificial

La interfaz y parte del código se desarrollaron con asistencia de Claude (Anthropic). El modelo biomecánico, las decisiones de diseño, la validación de los resultados y su interpretación son responsabilidad del grupo.

## Uso académico

Herramienta con fines educativos. No realiza diagnósticos.
