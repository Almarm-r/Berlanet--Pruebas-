# BeerLaNet — Normalización de tinciones basada en física

## Descripción

**BeerLaNet** es un método de aprendizaje profundo informado por la física para realizar **normalización adaptativa de tinciones en imágenes de microscopía**.

El método combina tres ideas principales:

1. **Ley de Beer–Lambert**, para modelar físicamente la absorción de la luz.
2. **Factorización matricial no negativa (NMF)**, para separar los componentes que contribuyen a la imagen.
3. **Desenrollado algorítmico (algorithmic unrolling)**, para convertir un procedimiento iterativo de optimización en una arquitectura entrenable.

La idea central es representar una imagen de microscopía no solamente como una matriz de valores RGB, sino como el resultado de un proceso físico:

```text
Fuente de iluminación
        │
        ▼
   Muestra biológica
        │
        ▼
 Absorción de la luz
        │
        ▼
 Imagen observada
```

Esto permite separar, de manera aproximada, la información relacionada con la **apariencia de la tinción** de la información asociada con la **distribución espacial de los componentes de la muestra**.

---

# 1. ¿Por qué es necesaria la normalización?

Las imágenes de microscopía pueden presentar diferencias de apariencia incluso cuando contienen estructuras biológicas similares.

Estas diferencias pueden producirse por:

* concentración de la tinción,
* protocolo de tinción,
* iluminación,
* microscopio,
* cámara,
* preparación de la muestra,
* condiciones de adquisición,
* diferencias entre laboratorios.

Por ejemplo, dos imágenes podrían contener células biológicamente similares:

```text
Imagen A                    Imagen B

misma estructura            misma estructura
      │                           │
      ▼                           ▼
tinción diferente            tinción diferente
      │                           │
      ▼                           ▼
RGB diferente                RGB diferente
```

Esto puede representar un problema para los modelos de aprendizaje automático.

Un clasificador podría aprender accidentalmente características relacionadas con:

> "cómo fue adquirida la imagen"

en lugar de características relacionadas con:

> "qué características biológicas contiene la imagen".

La normalización de tinciones busca reducir esta variabilidad no deseada.

---

# 2. Fundamento físico: ley de Beer–Lambert

BeerLaNet parte de la **ley de Beer–Lambert**, que describe la atenuación de la luz cuando atraviesa un material absorbente.

De manera simplificada:

$$
I = I_0 e^{-A}
$$

donde:

* \(I\) es la intensidad de luz observada,
* \(I_0\) es la intensidad de referencia o incidente,
* \(A\) representa la absorción acumulada.

Reorganizando:

$$
\frac{I}{I_0}=e^{-A}
$$

y aplicando el logaritmo:

$$
-\log\left(\frac{I}{I_0}\right)=A
$$

La cantidad:

$$
OD=-\log\left(\frac{I}{I_0}\right)
$$

se conoce como **densidad óptica (Optical Density, OD)**.

Este paso es importante porque la relación entre intensidad y absorción es exponencial en el espacio de intensidades, pero se vuelve aproximadamente lineal en el espacio de densidad óptica.

---

# 3. Del modelo físico a una imagen

Una imagen RGB puede representarse como:

$$
X \in \mathbb{R}^{C\times H\times W}
$$

donde:

* \(C\) = número de canales,
* \(H\) = altura,
* \(W\) = ancho.

Para una imagen RGB:

$$
C=3
$$

BeerLaNet busca representar la densidad óptica como una combinación de componentes:

$$
OD \approx SD
$$

donde:

* \(S\) representa las características espectrales/de color de los componentes absorbentes.
* \(D\) representa la distribución espacial o abundancia de esos componentes.

La idea puede visualizarse como:

```text
                 Imagen RGB
                     │
                     ▼
             Modelo Beer–Lambert
                     │
                     ▼
              Densidad óptica
                     │
                     ▼
                 S × D
                /     \
               /       \
              ▼         ▼
             S           D
       características   distribución
       de los componentes espacial
```

---

# 4. ¿Qué significa `X`?

En el código:

```python
X = transform(img).unsqueeze(0).to(device)

x0, S, D = beerlanet(
    X,
    S=None,
    D=None,
    n_iter=10,
    unit_norm_S=True
)
```

`X` es la **imagen observada que entra al modelo**.

Normalmente tiene la forma:

```text
[B, C, H, W]
```

donde:

* `B`: número de imágenes del lote (*batch*),
* `C`: canales,
* `H`: altura,
* `W`: ancho.

Para una imagen RGB individual:

```text
[B, C, H, W]
=
[1, 3, H, W]
```

Por lo tanto:

```text
X = imagen observada
```

Es importante entender que `X` todavía contiene mezclados todos los efectos presentes en la imagen:

```text
X
│
├── iluminación
├── tinción
├── estructuras celulares
├── fondo
└── variaciones de adquisición
```

BeerLaNet intenta obtener una representación más estructurada de estos componentes.

---

# 5. ¿Qué significa `x0`?

El modelo devuelve:

```python
x0
```

que está relacionado con la intensidad de referencia \(I_0\) de la ley de Beer–Lambert.

Recordemos:

$$
I = I_0e^{-A}
$$

Por lo tanto, conceptualmente:

```text
X  → imagen observada
x0 → estimación de la intensidad de referencia
```

La relación puede escribirse aproximadamente como:

$$
X \approx x_0\odot e^{-SD}
$$

donde \(\odot\) representa multiplicación elemento a elemento.

En consecuencia:

$$
-\log\left(\frac{X}{x_0}\right)
\approx SD
$$

Esto conecta directamente las variables del código con el modelo físico.

### Importante

`x0` no debe interpretarse simplemente como:

> "una versión mejorada de la imagen original".

Representa la estimación del componente de referencia/iluminación utilizado por el modelo para describir la formación de la imagen.

---

# 6. ¿Qué significa `S`?

`S` representa los **componentes espectrales o características de color de los componentes absorbentes**.

En términos intuitivos:

$$
\boxed{S=\text{qué componente es}}
$$

Por ejemplo, si la imagen contiene diferentes sustancias absorbentes asociadas a la tinción, cada una puede tener una firma diferente.

Conceptualmente:

```text
S

Componente 1 → firma espectral
Componente 2 → firma espectral
Componente 3 → firma espectral
...
```

Para una imagen RGB, estas firmas contienen información relacionada con la contribución de cada componente en los canales de color.

Una representación simplificada podría ser:

$$
S=
\begin{bmatrix}
s_{R1} & s_{R2} & \cdots\\
s_{G1} & s_{G2} & \cdots\\
s_{B1} & s_{B2} & \cdots
\end{bmatrix}
$$

Cada columna representa un componente absorbente.

Por lo tanto, `S` responde conceptualmente a:

> **¿Cómo se caracteriza espectralmente cada componente de la imagen?**

---

# 7. ¿Qué significa `D`?

`D` representa la **distribución espacial o abundancia de los componentes**.

Mientras que:

$$
S=\text{qué es el componente}
$$

tenemos:

$$
D=\text{dónde está y cuánto contribuye}
$$

Cada componente puede representarse como un mapa espacial:

$$
D_k(x,y)
$$

donde \(k\) identifica el componente y \((x,y)\) identifica la posición del píxel.

Conceptualmente:

```text
S
│
├── ¿Qué es el componente 1?
├── ¿Qué es el componente 2?
└── ¿Qué es el componente 3?

D
│
├── ¿Dónde está el componente 1?
├── ¿Dónde está el componente 2?
└── ¿Dónde está el componente 3?
```

---

# 8. Relación entre `S` y `D`

La relación más importante para entender BeerLaNet es:

$$
\boxed{S=\text{características espectrales}}
$$

$$
\boxed{D=\text{distribución espacial}}
$$

y juntos:

$$
\boxed{OD\approx SD}
$$

Es decir, BeerLaNet intenta explicar la densidad óptica de la imagen mediante una combinación de:

```text
          S                    D
          │                    │
          │                    │
   características        distribución
    de cada componente     de cada componente
          │                    │
          └─────────┬──────────┘
                    ▼
                   S·D
                    │
                    ▼
             Densidad óptica
```

---

# 9. ¿Por qué utilizar una factorización no negativa?

BeerLaNet utiliza una formulación relacionada con **Non-negative Matrix Factorization (NMF)**.

Se impone:

$$
S\geq0
$$

y

$$
D\geq0
$$

Esto tiene sentido físico porque las concentraciones o contribuciones de componentes absorbentes no deberían ser negativas.

Por ejemplo:

$$
OD=S_1D_1+S_2D_2+\cdots+S_KD_K
$$

representa una combinación de contribuciones positivas.

Esto contrasta con técnicas estadísticas como PCA, donde los componentes pueden tener coeficientes positivos y negativos.

La restricción de no negatividad proporciona una interpretación más cercana a una **mezcla física de componentes**.

---

# 10. Desenrollado algorítmico

Otro elemento fundamental de BeerLaNet es el **algorithmic unrolling**.

La factorización y estimación de los componentes puede plantearse como un problema de optimización iterativa.

Conceptualmente:

```text
Estimación inicial
       │
       ▼
 Iteración 1
       │
       ▼
 Iteración 2
       │
       ▼
 Iteración 3
       │
       ▼
      ...
       │
       ▼
 Iteración N
```

BeerLaNet convierte esta estructura iterativa en una arquitectura de red neuronal.

```text
Bloque 1
   │
   ▼
Bloque 2
   │
   ▼
Bloque 3
   │
   ▼
 ...
   │
   ▼
Bloque N
```

Cada bloque representa conceptualmente una actualización del proceso de optimización.

Esto permite combinar:

* conocimiento físico,
* estructura matemática,
* optimización iterativa,
* aprendizaje automático.

---

# 11. ¿Qué significa `n_iter`?

En tu código:

```python
n_iter=10
```

indica el número de iteraciones/bloques de actualización utilizados por el modelo.

Conceptualmente:

```text
X
│
▼
Inicialización
│
▼
Iteración 1
│
▼
Iteración 2
│
▼
...
│
▼
Iteración 10
│
▼
x0, S, D
```

Un número mayor de iteraciones permite realizar más actualizaciones, aunque esto **no significa automáticamente que el resultado sea mejor**.

El número óptimo debe determinarse experimentalmente para el conjunto de datos utilizado.

---

# 12. ¿Qué significa `S=None` y `D=None`?

En la llamada:

```python
S=None,
D=None
```

no se están proporcionando valores iniciales externos para estos parámetros.

BeerLaNet debe inicializarlos internamente y posteriormente actualizarlos durante las iteraciones.

Conceptualmente:

```text
S=None
  │
  ▼
Inicialización interna de S
  │
  ▼
Actualización
```

y:

```text
D=None
  │
  ▼
Inicialización interna de D
  │
  ▼
Actualización
```

Esto permite utilizar el modelo de manera adaptativa sobre nuevas imágenes sin tener que especificar manualmente una matriz de tinciones para cada una.

---

# 13. ¿Qué significa `unit_norm_S=True`?

El parámetro:

```python
unit_norm_S=True
```

indica que los componentes de `S` se normalizan.

Conceptualmente:

$$
\|S_k\|_2=1
$$

para cada componente \(k\).

Esto es importante porque la factorización:

$$
OD=SD
$$

tiene una ambigüedad de escala.

Por ejemplo:

$$
SD=(aS)\left(\frac{D}{a}\right)
$$

produce exactamente el mismo producto para un escalar positivo \(a\).

Por lo tanto, normalizar `S` ayuda a controlar esta ambigüedad.

### Importante

```python
unit_norm_S=True
```

**no significa que toda la imagen `X` se normalice a norma unitaria**.

La normalización se aplica a los componentes representados por `S`.

---

# 14. Interpretación completa de la llamada

La llamada:

```python
x0, S, D = beerlanet(
    X,
    S=None,
    D=None,
    n_iter=10,
    unit_norm_S=True
)
```

puede interpretarse como:

> Tomar la imagen observada `X`, estimar una intensidad de referencia `x0`, obtener los componentes espectrales/de tinción `S` y sus distribuciones espaciales `D`, y refinar estas estimaciones mediante 10 iteraciones del procedimiento desenrollado, normalizando los componentes de `S`.

Visualmente:

```text
                     X
                     │
                     │
              Imagen observada
                     │
                     ▼
              ┌─────────────┐
              │  BeerLaNet  │
              └─────────────┘
                     │
          ┌──────────┼──────────┐
          │          │          │
          ▼          ▼          ▼
         x0          S          D
          │          │          │
          │          │          │
          │          │          └── Distribución espacial
          │          └───────────── Componentes espectrales
          └──────────────────────── Intensidad de referencia
```

---

# 15. Modelo matemático completo

La relación conceptual entre las variables puede expresarse como:

$$
X\approx x_0\odot e^{-SD}
$$

Dividiendo por \(x_0\):

$$
\frac{X}{x_0}\approx e^{-SD}
$$

Aplicando el logaritmo negativo:

$$
-\log\left(\frac{X}{x_0}\right)\approx SD
$$

Por tanto:

$$
\boxed{
OD\approx SD
}
$$

Esta ecuación resume la conexión entre:

* la imagen observada,
* la ley de Beer–Lambert,
* la densidad óptica,
* los componentes espectrales,
* y las distribuciones espaciales.

---

# 16. ¿Por qué puede ser útil para imágenes de malaria?

En imágenes de microscopía de malaria podemos tener:

```text
              Variación observada
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   tinción       iluminación    adquisición
       │             │             │
       └─────────────┼─────────────┘
                     ▼
                Imagen RGB
```

Dos células biológicamente similares pueden presentar valores RGB diferentes debido a estas condiciones.

El problema es que un modelo de clasificación podría aprender esas diferencias de apariencia como si fueran diferencias biológicas.

BeerLaNet proporciona una representación intermedia:

```text
                 Imagen RGB
                     │
                     ▼
               BeerLaNet
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
          S                     D
  componentes de tinción   distribución espacial
          │                     │
          └──────────┬──────────┘
                     ▼
            Representación
              normalizada
                     │
                     ▼
              Clasificador
                     │
              ┌──────┴──────┐
              ▼             ▼
           Sano         Infectado
```

La hipótesis experimental es que una representación menos dependiente de las condiciones de adquisición puede ayudar al modelo posterior a concentrarse en características morfológicas relevantes.

---

# 17. ¿Por qué podría ser mejor que una normalización RGB tradicional?

Una normalización convencional puede trabajar directamente sobre los valores de los canales:

```text
R ──► normalización
G ──► normalización
B ──► normalización
```

Este procedimiento puede corregir diferencias estadísticas entre imágenes, pero no necesariamente modela el proceso físico que produjo esos valores.

BeerLaNet introduce:

```text
RGB
 │
 ▼
Modelo de Beer–Lambert
 │
 ▼
Densidad óptica
 │
 ▼
Factorización no negativa
 │
 ├── S → componentes espectrales
 └── D → distribución espacial
```

Por esta razón, su principal diferencia no es simplemente que "cambie mejor los colores", sino que incorpora una **hipótesis física sobre la formación de la imagen**.

---

# 18. BeerLaNet y CIELAB

BeerLaNet está formulado a partir de intensidades de luz y densidad óptica.

La transformación física es:

$$
OD=-\log\left(\frac{I}{I_0}\right)
$$

donde \(I\) representa una intensidad de luz.

CIELAB, en cambio, es un espacio de color perceptual:

$$
RGB\rightarrow XYZ\rightarrow L^*a^*b^*
$$

Los canales \(L^*\), \(a^*\) y \(b^*\) no tienen la misma interpretación física que los canales de intensidad RGB.

Por esta razón, una tubería físicamente consistente sería:

```text
RGB original
     │
     ▼
  BeerLaNet
     │
     ▼
RGB normalizado
     │
     ▼
   CIELAB
     │
     ▼
Análisis / ML
```

y no necesariamente:

```text
RGB
 │
 ▼
CIELAB
 │
 ▼
BeerLaNet
```

si se quiere conservar la interpretación de Beer–Lambert.

---

# 19. Resumen de variables

| Variable      | Significado                        | Interpretación                                  |
| ------------- | ---------------------------------- | ----------------------------------------------- |
| `X`           | Imagen observada                   | Entrada RGB                                     |
| `x0`          | Intensidad de referencia estimada  | Relacionada con \(I_0\)                         |
| `S`           | Componentes espectrales/de tinción | Describe las características de cada componente |
| `D`           | Distribución espacial              | Describe dónde y cuánto aparece cada componente |
| `S @ D`       | Densidad óptica estimada           | Combinación espectral + espacial                |
| `n_iter`      | Número de iteraciones              | Número de actualizaciones del modelo            |
| `unit_norm_S` | Normalización de `S`               | Controla la ambigüedad de escala                |

---

