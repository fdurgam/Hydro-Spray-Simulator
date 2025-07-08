
# El Aspersor como un Cañón de Agua: Entendiendo el Radio de Riego de Hydro Spray Simulator
Imaginemos que el aspersor no lanza agua, sino una pequeña bala de agua. El simulador lo que hace es calcular el camino que recorrerá esa bala desde que sale del cañón hasta que cae al suelo.
Para saber qué tan lejos caerá (**el radio de riego**), el simulador debe obtener cierta infomación Clave:

1. ¿Con qué fuerza dispara el cañón? (Presión → Velocidad).
   
    Le ponemos presión al agua, y eso le da una velocidad inicial al chorro. A más presión, más rápido sale el agua.

    **Visualízación** : Si el agua sale con poca fuerza, no llegará lejos. Si sale con mucha fuerza, volará más lejos.

2. ¿Hacia dónde apunta el cañón? (Ángulo y Altura)
   
   El cañón no apunta recto, sino un poco hacia arriba. También está a una altura del suelo. El simulador calcula qué parte de esa fuerza inicial va hacia adelante y qué parte va hacia arriba.
    
    **Visualízación**: Un cañón que dispara muy plano no cubre mucha distancia. Uno que apunta muy alto hace que el agua suba mucho, pero no necesariamente lejos horizontalmente. La clave es el **ángulo** y la **altura** desde donde se lanza.

3. ¿Cuánto tiempo tarda la bala en caer? (Tiempo de Vuelo)
    Una vez que la bala de agua sale, la gravedad empieza a ejercer preción sobre ella hacia abajo. El simulador calcula cuánto tiempo pasa desde que el agua sale hasta que toca el suelo.

    **Visualízación**: Si el agua sale con mucha altura, tardará más en caer. Si sale con poca, caerá mas rápido.

4. ¿Qué tan lejos llega el agua? (Radio de Riego)
   
   Cuando ya sabemos cuánto tiempo estuvo la bala en el aire y qué tan rápido se movía hacia adelante, simplemente multiplicamos esos dos valores para saber la distancia total que recorrió en línea recta sobre el suelo. ¡Esa es la distancia máxima que riega el aspersor a una presión constante, **el radio de riego**!

    **Visualízación**: La distancia desde la base del aspersor hasta el punto más lejano donde el agua aterrizó.

# Las Matemáticas del Radio de Riego en Hydro Spray Simulator 
El radio de riego no se calcula de forma directa a partir de la presión. En realidad, es el resultado final de una **secuencia de cálculos físicos** que simulan el comportamiento real del agua como un proyectil. La presión inicial es el punto de partida, y a partir de ella, determinamos la velocidad del chorro, la trayectoria parabólica, y finalmente, el alcance horizontal que define el radio de riego del aspersor.

---
## 1. De la Presión a la Velocidad Inicial del Agua
Traducimos la presión del agua en la velocidad con la que el chorro sale de la boquilla, utilizaando la **relación lineal simple**: más presión significa más velocidad.

$$v_{\text{inicial}} = \left( \frac{P_{\text{actual}}}{P_{\text{máxima}}} \right) \times v_{\text{máxima\_inicial}}$$

* $v_{\text{inicial}}$: Es la **velocidad inicial** del agua al salir de la boquilla (en metros por segundo, m/s).
* $P_{\text{actual}}$: Es la **presión actual** configurada en PSI (**NUESTRA VARIABLE INDEPENDIENTE**).
* $P_{\text{máxima}}$: Es la **presión máxima** que puede alcanzar el sistema (100 PSI). (**CONSTANTE que definimos nosotros**)
* $v_{\text{máxima\_inicial}}$: Es la **velocidad inicial máxima** que el agua puede lograr a la presión máxima (25 m/s). <span style="color:red">Esto esta al la bibliografica son alrrededor de 70 km/h, pero podriamos buscar este dato, es una constante, </span>


**Ejemplo:**
Dada la **presión actual** en **50 PSI**:
$$v_{\text{inicial}} = \left( \frac{50 \, \text{PSI}}{100 \, \text{PSI}} \right) \times 25 \, \text{m/s} = 0.5 \times 25 \, \text{m/s} = 12.5 \, \text{m/s}$$

---

## 2. Descomposición de la Velocidad y Altura de Lanzamiento
El agua sale disparada en una dirección específica (generalmente con un ángulo hacia arriba) de la boquilla. Para calcular su trayectoria, es necesario conocer qué tan rápido se mueve **horizontalmente** y qué tan rápido se mueve **verticalmente**. También es crucial conocer la **altura desde la que se lanza** el chorro.
* **Vector unitario de dirección de lanzamiento ($\hat{d}$):** Obtenemos la  orientación de la boquilla (`sprinkler_nozzle_live.axis.norm()`). Este vector indica  la dirección precisa en la que el agua empieza a moverse.
* **Velocidad inicial horizontal ($v_{xz}$):** Es la parte de la velocidad que impulsa el agua a lo largo del suelo.
    $$v_{xz} = v_{inicial} \times \text{magnitud}(\text{vector}(\hat{d}.x, 0, \hat{d}.z))$$

* **Velocidad inicial vertical ($v_y$):** Esta es la parte de la velocidad que lanza el agua hacia arriba.
    $$v_y = v_{inicial} \times \hat{d}.y$$

* **Altura de lanzamiento ($y_0$):** Es la altura de la boquilla sobre el suelo en el momento del lanzamiento.
    $$y_0 = \text{sprinkler\_nozzle\_live.pos.y}$$

**Ejemplo:**
Asumamos que la boquilla apunta con una dirección unitaria $\hat{d}$ de `(0.707, 0.707, 0)` (esto significa que se lanza a unos 45 grados hacia arriba, asumiendo un movimiento en el plano XZ).
Con $v_{inicial} = 12.5 \, \text{m/s}$:
* La magnitud horizontal de $\hat{d}$ es $\text{magnitud}(\text{vector}(0.707, 0, 0)) = 0.707$.
    $$v_{xz} = 12.5 \, \text{m/s} \times 0.707 \approx 8.84 \, \text{m/s}$$
* La componente vertical de $\hat{d}$ es $0.707$.
    $$v_y = 12.5 \, \text{m/s} \times 0.707 \approx 8.84 \, \text{m/s}$$

Si la **altura de la boquilla** sobre el suelo (`sprinkler_nozzle_live.pos.y`) es **1.0 metro**, entonces $y_0 = 1.0 \, \text{m}$.

---

## 3. Cálculo del Tiempo de Vuelo (Cuando el Agua Toca el Suelo)

Es la **cinemática de proyectiles**. Queremos saber cuánto tiempo permanece el agua en el aire antes de caer al suelo. Usamos la ecuación de movimiento vertical bajo la influencia de la gravedad:

La altura $y(t)$ en cualquier momento $t$ se describe como:
$$y(t) = y_0 + v_y t - \frac{1}{2} g t^2$$

Donde $g$ es la aceleración debido a la gravedad (aproximadamente 9.8 m/s²).

Para encontrar el **tiempo de vuelo**, establecemos $y(t) = 0$ (cuando el agua llega al suelo) y reorganizamos la ecuación en una **forma cuadrática** estándar ($at^2 + bt + c = 0$):
$$\left( \frac{1}{2} g \right) t^2 - v_y t - y_0 = 0$$

Luego, aplicamos la **fórmula cuadrática** para resolver $t$:
$$t = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$

Sustituyendo los valores de $a = \frac{1}{2} g$, $b = -v_y$, y $c = -y_0$:
$$t = \frac{-(-v_y) \pm \sqrt{(-v_y)^2 - 4(\frac{1}{2} g)(-y_0)}}{2(\frac{1}{2} g)}$$

Esto se simplifica a la fórmula (tomando solo la solución positiva para el tiempo):
$$\text{tiempo\_de\_vuelo} = \frac{v_y + \sqrt{v_y^2 + 2 g y_0}}{g}$$

**Ejemplo:**
Con $v_y = 8.84 \, \text{m/s}$, $y_0 = 1.0 \, \text{m}$, y $g = 9.8 \, \text{m/s}^2$:

1.  **Calculamos el Discriminante** (la parte bajo la raíz cuadrada):
    $$\text{Discriminante} = v_y^2 + 2 g y_0 = (8.84)^2 + (2 \times 9.8 \times 1.0) = 78.14 + 19.6 = 97.74$$

2.  **Calculamos el Tiempo de Vuelo:**
    $$\text{tiempo\_de\_vuelo} = \frac{8.84 + \sqrt{97.74}}{9.8}\text= \frac{8.84 + 9.886}{9.8} = \frac{18.726}{9.8} \approx 1.91 \, \text{segundos}$$

---

## 4. Cálculo del Alcance Horizontal (El Radio de Riego)

Una vez que sabemos cuánto tiempo está el agua en el aire (`tiempo_de_vuelo`), podemos calcular qué tan lejos se mueve horizontalmente. Asumimos que la velocidad horizontal es constante (ignorando la resistencia del aire para simplificar la simulación).

$$\text{Alcance} = v_{xz} \times \text{tiempo\_de\_vuelo}$$

**Ejemplo:**
Con $v_{xz} = 8.84 \, \text{m/s}$ y $\text{tiempo\_de\_vuelo} = 1.91 \, \text{segundos}$:
$$\text{Alcance} = 8.84 \, \text{m/s} \times 1.91 \, \text{s} \approx 16.89 \, \text{metros}$$

Este valor de "Alcance" es exactamente lo que asigna a `water_spread_radius` y por lo tanto, es el **radio de riego** que se muestra visualmente en el círculo azul en el suelo y en las etiquetas de la interfaz.
## 5. Conclusión
El Hydro Spray Simulator transforma la presión en un radio de riego a través de una serie de cálculos que simulan la física de un proyectil de agua, considerando la velocidad inicial, la dirección de lanzamiento, la altura y el efecto de la gravedad.


## Referencias
[Movimiento de Proyectiles](https://espanol.libretexts.org/Fisica/Libro%3A_Fisica_Basada_en_Calculo_(Schnick)/Volumen_A%3A_Cin%C3%A9tica%2C_Est%C3%A1tica_y_Termodin%C3%A1mica/13A%3A_ca%C3%ADda_libre%2C_tambi%C3%A9n_conocido_como_Movimiento_de_Proyectiles#:~:text=Esto%20significa%20que%20si%20disparas,y%20movimiento%20es%20el%20tiempo.
