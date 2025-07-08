
\documentclass{article}
\usepackage[utf8]{inputenc}
\usepackage{amsmath} % Para entornos matemáticos como align y gathers
\usepackage{amssymb} % Para símbolos matemáticos adicionales
\usepackage{graphicx} % Si quisieras incluir imágenes (no en este caso)
\usepackage{geometry} % Para configurar márgenes
\geometry{a4paper, margin=1in}



## Las Matemáticas del Radio de Riego en tu Simulador
\section*{Introducción}
El radio de riego de tu aspersor no se calcula de forma directa y mágica a partir de la presión. En realidad, es el resultado final de una **secuencia de cálculos físicos** que simulan el comportamiento real del agua. La presión inicial es el punto de partida, y a partir de ella, se determinan la velocidad del chorro, su trayectoria parabólica, y finalmente, el alcance horizontal que define el radio.

Vamos a desglosar este proceso paso a paso, incluyendo las fórmulas implícitas que usa tu código y un ejemplo para que sea más fácil de entender

---

## De la Presión a la Velocidad Inicial del Agua
El primer paso es traducir la presión del agua en la velocidad con la que el chorro sale de la boquilla. Tu código utiliza una relación lineal simple: más presión significa más velocidad.




$$v_{\text{inicial}} = \left( \frac{P_{\text{actual}}}{P_{\text{máxima}}} \right) \times v_{\text{máxima\_inicial}}$$


$$v_{\text{inicial}} = \left( \frac{50 \, \text{PSI}}{100 \, \text{PSI}} \right) \times 25 \, \text{m/s} = 0.5 \times 25 \, \text{m/s} = 12.5 \, \text{m/s}$$




Si ajustas la **presión actual** a **50** PSI:
$$v_{\text{inicial}} = \left( \frac{50 \, \text{PSI}}{100 \, \text{PSI}} \right) \times 25 \, \text{m/s} = 0.5 \times 25 \, \text{m/s} = 12.5 \, \text{m/s}$$

---

\section{Descomposición de la Velocidad y Altura de Lanzamiento}
El agua no sale disparada solo hacia adelante, sino en una dirección específica (generalmente con un ángulo hacia arriba) definida por la boquilla. Para calcular su trayectoria, necesitamos saber qué tan rápido se mueve \textbf{horizontalmente} y qué tan rápido se mueve \textbf{verticalmente}. También es crucial conocer la \textbf{altura desde la que se lanza} el chorro.

\begin{itemize}
    \item \textbf{Vector unitario de dirección de lanzamiento ($\hat{d}$):} Tu código obtiene esto de la orientación de la boquilla (\texttt{sprinkler\_nozzle\_live.axis.norm()}). Este vector nos dice la dirección precisa en la que el agua empieza a moverse.
    \item \textbf{Velocidad inicial horizontal ($v_{xz}$):} Esta es la parte de la velocidad que impulsa el agua a lo largo del suelo.
    $$v_{xz} = v_{\text{inicial}} \times \text{magnitud}(\text{vector}(\hat{d}.x, 0, \hat{d}.z))$$
    \item \textbf{Velocidad inicial vertical ($v_y$):} Esta es la parte de la velocidad que lanza el agua hacia arriba.
    $$v_y = v_{\text{inicial}} \times \hat{d}.y$$
    \item \textbf{Altura de lanzamiento ($y_0$):} Esta es la altura de la boquilla sobre el suelo en el momento del lanzamiento.
    $$y_0 = \text{sprinkler\_nozzle\_live.pos.y}$$
\end{itemize}

\textbf{Ejemplo (Continuación):} \\
Asumamos que la boquilla apunta con una dirección unitaria $\hat{d}$ de $(0.707, 0.707, 0)$ (esto significa que se lanza a unos 45 grados hacia arriba, asumiendo un movimiento en el plano XZ).
Con $v_{\text{inicial}} = 12.5 \, \text{m/s}$:
\begin{itemize}
    \item La magnitud horizontal de $\hat{d}$ es $\text{magnitud}(\text{vector}(0.707, 0, 0)) = 0.707$.
    $$v_{xz} = 12.5 \, \text{m/s} \times 0.707 \approx 8.84 \, \text{m/s}$$
    \item La componente vertical de $\hat{d}$ es $0.707$.
    $$v_y = 12.5 \, \text{m/s} \times 0.707 \approx 8.84 \, \text{m/s}$$
\end{itemize}
Si la \textbf{altura de la boquilla} sobre el suelo (\texttt{sprinkler\_nozzle\_live.pos.y}) es \textbf{1.0 metro}, entonces $y_0 = 1.0 \, \text{m}$.

---

\section{Cálculo del Tiempo de Vuelo (Cuando el Agua Toca el Suelo)}
Este es el corazón de la \textbf{cinemática de proyectiles}. Queremos saber cuánto tiempo permanece el agua en el aire antes de caer al suelo. Usamos la ecuación de movimiento vertical bajo la influencia de la gravedad:

La altura $y(t)$ en cualquier momento $t$ se describe como:
$$y(t) = y_0 + v_y t - \frac{1}{2} g t^2$$

Donde $g$ es la aceleración debido a la gravedad (aproximadamente 9.8 m/s²).

Para encontrar el \textbf{tiempo de vuelo}, establecemos $y(t) = 0$ (cuando el agua llega al suelo) y reorganizamos la ecuación en una \textbf{forma cuadrática} estándar ($at^2 + bt + c = 0$):
$$\left( \frac{1}{2} g \right) t^2 - v_y t - y_0 = 0$$

Luego, aplicamos la \textbf{fórmula cuadrática} para resolver $t$:
$$t = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}$$

Sustituyendo los valores de $a = \frac{1}{2} g$, $b = -v_y$, y $c = -y_0$:
$$t = \frac{-(-v_y) \pm \sqrt{(-v_y)^2 - 4(\frac{1}{2} g)(-y_0)}}{2(\frac{1}{2} g)}$$

Esto se simplifica a la fórmula que usa tu código (tomando solo la solución positiva para el tiempo):
$$\text{tiempo\_de\_vuelo} = \frac{v_y + \sqrt{v_y^2 + 2 g y_0}}{g}$$

\textbf{Ejemplo (Continuación):} \\
Con $v_y = 8.84 \, \text{m/s}$, $y_0 = 1.0 \, \text{m}$, y $g = 9.8 \, \text{m/s}^2$:
\begin{enumerate}
    \item \textbf{Calculamos el Discriminante} (la parte bajo la raíz cuadrada):
    $$\text{Discriminante} = v_y^2 + 2 g y_0 = (8.84)^2 + (2 \times 9.8 \times 1.0)$$
    $$\text{Discriminante} = 78.14 + 19.6 = 97.74$$
    \item \textbf{Calculamos el Tiempo de Vuelo:}
    $$\text{tiempo\_de\_vuelo} = \frac{8.84 + \sqrt{97.74}}{9.8}$$
    $$\text{tiempo\_de\_vuelo} = \frac{8.84 + 9.886}{9.8} = \frac{18.726}{9.8} \approx 1.91 \, \text{segundos}$$
\end{enumerate}

---

\section{Cálculo del Alcance Horizontal (El Radio de Riego)}
Una vez que sabemos cuánto tiempo está el agua en el aire (\texttt{tiempo\_de\_vuelo}), podemos calcular qué tan lejos se mueve horizontalmente. Asumimos que la velocidad horizontal es constante (ignorando la resistencia del aire para simplificar la simulación).

$$\text{Alcance} = v_{xz} \times \text{tiempo\_de\_vuelo}$$

\textbf{Ejemplo (Continuación):} \\
Con $v_{xz} = 8.84 \, \text{m/s}$ y $\text{tiempo\_de\_vuelo} = 1.91 \, \text{segundos}$:
$$\text{Alcance} = 8.84 \, \text{m/s} \times 1.91 \, \text{s} \approx 16.89 \, \text{metros}$$

Este valor de "Alcance" es exactamente lo que tu código asigna a \texttt{water\_spread\_radius} y, por lo tanto, es el \textbf{radio de riego} que se muestra visualmente en el círculo azul del suelo y en las etiquetas de la interfaz.

---

\section*{Conclusión}
En resumen, la presión no se convierte directamente en un radio; en cambio, es el punto de inicio de un cálculo físico que simula el movimiento del agua como un proyectil, y el alcance de ese proyectil es lo que finalmente define el radio de riego de tu aspersor.

\end{document}