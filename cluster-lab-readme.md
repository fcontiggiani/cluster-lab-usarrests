# 🔬 Laboratorio interactivo de Análisis de Clúster — USArrests

Herramienta educativa autocontenida (HTML + JavaScript + D3.js) para explorar de manera interactiva los conceptos centrales del **análisis de clúster** y del **análisis de componentes principales (PCA)**, utilizando el clásico dataset *USArrests* (50 estados de EE. UU., variables: `Murder`, `Assault`, `UrbanPop`, `Rape`).

No requiere instalación, backend ni dependencias externas más allá de D3.js (cargado vía CDN). Todo el cómputo estadístico —covarianzas, correlaciones, distancias, K-Means, clustering jerárquico, silueta, índice de Calinski–Harabasz y PCA por descomposición espectral (algoritmo de Jacobi)— está implementado desde cero en JavaScript puro.

---

## 1. Fundamentos teóricos

### 1.1 Estandarización
Las variables de *USArrests* están en escalas distintas (tasas por 100.000 habitantes vs. porcentaje de población urbana). El checkbox **"Estandarizar (z-score)"** transforma cada variable $x_j$ a:

$$z_{ij} = \frac{x_{ij} - \bar{x}_j}{s_j}$$

Cuando los datos están estandarizados, la matriz de covarianzas coincide con la matriz de correlaciones.

### 1.2 Matrices de covarianza y correlación
Se calculan según las definiciones muestrales habituales:

$$\text{Cov}(X_i,X_j) = \frac{1}{n-1}\sum_{k=1}^n (x_{ki}-\bar{x}_i)(x_{kj}-\bar{x}_j), \qquad
\text{Cor}(X_i,X_j) = \frac{\text{Cov}(X_i,X_j)}{s_i \, s_j}$$

El **heatmap** visualiza la matriz de correlaciones con una escala divergente (rojo = correlación negativa, azul = positiva).

### 1.3 Medidas de distancia entre observaciones
Sobre el vector de 4 variables de cada estado se pueden calcular seis métricas de (di)similitud:

| Medida | Fórmula conceptual |
|---|---|
| Euclídea | $\sqrt{\sum_j (x_j-y_j)^2}$ |
| Manhattan | $\sum_j |x_j-y_j|$ |
| 1 − Pearson | $1-r_{xy}$ |
| 1 − Spearman | $1-r_{\text{rank}(x),\text{rank}(y)}$ |
| 1 − Kendall (τ-b) | basada en pares concordantes/discordantes, con corrección por empates |
| Eisen (cosenoidal) | $1-\dfrac{\sum_j x_jy_j}{\|x\|\,\|y\|}$ (similitud coseno no centrada, usual en bioinformática) |

Estas distancias alimentan el **dendrograma** y los **índices de validación de clústers**.

### 1.4 Número óptimo de clústers
Se calculan tres criterios para $k=1..8$ (o $k=2..8$ cuando aplica), usando K-Means con 10 inicializaciones *k-means++* y semilla fija (reproducible):

- **Método del codo (WCSS)**: suma de cuadrados intra-clúster; se busca el "quiebre" en la curva.
- **Coeficiente de silueta promedio**: $s(i)=\dfrac{b(i)-a(i)}{\max(a(i),b(i))}$, donde $a(i)$ es la distancia promedio al propio clúster y $b(i)$ al clúster vecino más cercano. Valores cercanos a 1 indican mejor separación.
- **Índice de Calinski–Harabasz**: $\text{CH}=\dfrac{\text{BSS}/(k-1)}{\text{WSS}/(n-k)}$, razón entre dispersión inter e intra-clúster (mayor es mejor).

El valor de $k$ elegido en el control superior se resalta en los tres gráficos.

### 1.5 K-Means
Algoritmo de Lloyd con inicialización *k-means++*, 100 iteraciones máximas y selección de la mejor de 10 corridas según WCSS.

### 1.6 Clustering jerárquico aglomerativo
Construye un árbol binario fusionando iterativamente los dos clústers más cercanos, actualizando las distancias mediante la **fórmula de Lance–Williams**. Se ofrecen cuatro criterios de enlace:

- **Simple** (mínimo)
- **Completo** (máximo)
- **Promedio (UPGMA)**
- **Ward** (minimiza el incremento de varianza intra-clúster)

El **dendrograma** se corta automáticamente en $k$ grupos: las fusiones por encima del corte se dibujan en gris, y cada subárbol resultante se colorea según su clúster.

### 1.7 PCA y espacio de observaciones
Se calcula la matriz de covarianzas de los datos activos (estandarizados o no) y se obtiene su descomposición espectral mediante el **algoritmo de Jacobi** (rotaciones sucesivas hasta diagonalizar). Los dos primeros componentes principales (mayor varianza explicada) definen el plano de proyección. Cada observación se colorea según su clúster (K-Means o jerárquico, según corresponda) y se dibuja la **envolvente convexa** de cada grupo como región de clúster.

---

## 2. Instructivo de uso

1. **Estandarizar variables**: tildar/destildar el checkbox recalcula matrices, distancias, dendrograma y PCA en cascada.
2. **Medida de distancia**: elegir entre las seis opciones disponibles; afecta al dendrograma, al silhouette y al índice CH.
3. **Método de clustering**:
   - *K-Means*: usa directamente los datos (no la matriz de distancias) con distancia euclídea.
   - *Jerárquico*: habilita el selector de **enlace** (linkage) y muestra el dendrograma.
4. **Slider k**: define el número de clústers (2 a 10). Se refleja en el corte del dendrograma, las regiones del PCA y el marcador en los gráficos de número óptimo.
5. **Tooltips**: pasar el mouse sobre los puntos del PCA o sobre los puntos de los gráficos de k óptimo para ver valores exactos y nombres de los estados.

---

## 3. Despliegue en GitHub Pages

1. Crear un repositorio nuevo (público) en GitHub, por ejemplo `cluster-lab-usarrests`.
2. Agregar el archivo HTML del laboratorio con el nombre `index.html` en la raíz del repositorio (o dentro de una carpeta `/docs`).
3. Agregar este archivo como `README.md` en la raíz.
4. Ir a **Settings → Pages**, y en *Source* seleccionar la rama (`main`) y la carpeta (`/root` o `/docs` según corresponda).
5. Guardar; GitHub generará una URL del tipo `https://<usuario>.github.io/cluster-lab-usarrests/`.
6. La página quedará disponible públicamente y se actualizará automáticamente con cada `push` a la rama configurada.

---

## 4. Estructura del repositorio sugerida

```
cluster-lab-usarrests/
├── index.html      # Laboratorio interactivo (HTML + JS + D3, autocontenido)
└── README.md        # Este archivo
```

---

## 5. Créditos y referencias

- Dataset: *USArrests* (McNeil, D. R., 1977; incluido en el paquete base `datasets` de R).
- Visualización: [D3.js](https://d3js.org/) v7.
- Implementaciones propias: covarianza/correlación, distancias (Euclídea, Manhattan, Pearson, Spearman, Kendall τ-b, Eisen), K-Means (k-means++), clustering jerárquico (Lance–Williams), silueta, Calinski–Harabasz y PCA (algoritmo de Jacobi).
