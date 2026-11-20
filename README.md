# Análisis de afinidad e intereses en comunidades de redes sociales (`analisis-afinidad-hobbies`)

> **Pregunta de investigación:** ¿Qué gustos e intereses suelen compartir las comunidades de usuarios en redes sociales?

Este repositorio contiene el desarrollo completo de un proyecto académico para la asignatura de **Computación Social y Personalización** (Grado en Ciencia de Datos). El objetivo es modelar, detectar y analizar comunidades de usuarios a partir de sus afinidades temáticas, integrando técnicas de **Análisis de Redes Sociales (SNA)**, ingeniería de datos en Python, extracción semántica mediante **Modelos de Lenguaje (LLM)** y visualización topológica en **Gephi**.

---

## Tabla de contenidos

1. [Contexto y planteamiento del problema](#contexto-y-planteamiento-del-problema)
2. [Conjunto de datos](#conjunto-de-datos)
3. [Metodología y pipeline de procesamiento](#metodología-y-pipeline-de-procesamiento)
4. [Resultados y análisis de la red](#resultados-y-análisis-de-la-red)
   - [Métricas topológicas globales](#métricas-topológicas-globales)
   - [Caracterización de las comunidades detectadas](#caracterización-de-las-comunidades-detectadas)
   - [Conectividad intercomunitaria y conclusiones](#conectividad-intercomunitaria-y-conclusiones)
5. [Estructura del repositorio](#estructura-del-repositorio)
6. [Requisitos e instalación](#requisitos-e-instalación)
7. [Reproducibilidad y ejecución](#reproducibilidad-y-ejecución)
8. [Autores y licencia](#autores-y-licencia)

---

## Contexto y planteamiento del problema

En el estudio clásico de redes sociales, la estructura topológica suele construirse a partir de interacciones explícitas entre cuentas (seguimientos, menciones o retweets). Sin embargo, al muestrear un subconjunto reducido de redes ego-céntricas (*ego-networks*) por restricciones computacionales, el uso directo de la lista de aristas originales (`.edges`) genera **componentes disconexos y comunidades aisladas**, lo que impide estudiar la cohesión global de la muestra.

Para abordar esta limitación, se propone un modelo de **grafo no dirigido y ponderado basado en similitud de intereses** (equivalente a una proyección unipartita de la red bipartita usuario–interés):

- **Nodos ($V$):** Usuarios únicos de Twitter con al menos un interés compartido.
- **Aristas ($E$):** Relación no dirigida entre dos usuarios si existe intersección no vacía entre sus conjuntos de intereses.
- **Pesos ($w_{ij}$):** Cardinalidad de la intersección de intereses compartidos entre los usuarios $i$ y $j$ ($w_{ij} = |I_i \cap I_j|$).

Este planteamiento garantiza una mayor conectividad estructural y permite cuantificar la fuerza del vínculo temático entre perfiles.

---

## Conjunto de datos

Los datos base se han obtenido del repositorio **SNAP (Stanford Network Analysis Project)** de la Universidad de Stanford:

- **Fuente:** [Social Circles: Twitter (`ego-Twitter`)](https://snap.stanford.edu/data/ego-Twitter.html)
- **Archivos empleados del dataset original:**
  - `.feat` y `.egofeat`: Matrices binarias que codifican la presencia o ausencia de características (hashtags y menciones) para los nodos del círculo y el nodo central (*ego*).
  - `.featnames`: Tabla de correspondencia entre cada índice de columna y el término real asociado.

---

## Metodología y pipeline de procesamiento

El flujo de trabajo se estructura en cuatro etapas secuenciales:

### 1. Extracción y agregación de atributos (`1-user_hashtags_creation.ipynb`)
- Por limitaciones de capacidad de cómputo y cuota de procesamiento en la etapa de inferencia con LLM, se seleccionó una muestra inicial correspondiente a **10 *ego-networks*** del dataset de Stanford.
- Se cruzaron los archivos `.feat`, `.egofeat` y `.featnames` para reconstruir el conjunto de hashtags y menciones de cada usuario.
- Se eliminaron duplicados agrupando por identificador de usuario y se exportó el dataset intermedio `user_hashtags.csv`.

### 2. Enriquecimiento semántico mediante LLM
- Los hashtags y menciones en bruto presentan un alto nivel de ruido, abreviaturas y dispersión léxica. Para homogeneizar el espacio de características, se empleó el modelo de lenguaje **DeepSeek** con el fin de inferir y categorizar los intereses y aficiones subyacentes de cada usuario.
- El resultado se consolidó en `user_interests.csv`. Tras la limpieza de cadenas, normalización a minúsculas y descarte de registros con valores nulos o insuficientes, se conservaron **793 usuarios** válidos.

### 3. Modelado del grafo y detección de comunidades (`2-analisis_redsocial_twitter.ipynb`)
- Se construyó el grafo ponderado en **NetworkX** evaluando la intersección de intereses por pares de usuarios y se depuraron los nodos aislados, resultando en una red conexa de **789 nodos**.
- Se aplicó el **algoritmo de Louvain** (`community_louvain.best_partition`) optimizando la modularidad ponderada (`weight='weight'`).
- Para interpretar semánticamente los clústeres hallados, se recopilaron los intereses agregados por comunidad y se sintetizaron sus categorías representativas mediante un LLM.
- Se computaron métricas de centralidad y topología global (densidad, coeficiente de clustering promedio ponderado, componentes conexas, radio y diámetro) y se exportó la estructura en formato GEXF (`red_social.gexf`).

### 4. Visualización y síntesis de resultados
- Se generaron gráficos exploratorios y descriptivos con `Matplotlib` (diagrama de sectores por tamaño relativo, barras de aristas intercomunitarias y grafo agregado de comunidades).
- Se elaboró el grafo final de alta densidad en **Gephi** (`RED_GEPHI.gephi`) y se consolidó el resumen ejecutivo del análisis en la infografía `Resultados.pdf`.

---

## Resultados y análisis de la red

### Métricas topológicas globales

| Métrica | Valor | Interpretación analítica |
| :--- | :---: | :--- |
| **Nodos ($|V|$)** | `789` | Usuarios con al menos un interés compartido tras eliminar nodos aislados. |
| **Aristas ($|E|$)** | `86,199` | Alto volumen de vínculos de afinidad dentro de la muestra. |
| **Componentes conexas** | `1` | Red completamente conexa; no existen subgrafos aislados. |
| **Densidad** | `0.2772` | Conectividad elevada para una red social de este orden, propia de proyecciones por intereses. |
| **Coeficiente de clustering (ponderado)** | `0.2877` | Tendencia moderada a la formación de tríadas y grupos cohesionados temáticamente. |
| **Radio** | `3` | Excentricidad mínima en el grafo. |
| **Diámetro** | `5` | Distancia geodésica máxima reducida entre cualquier par de usuarios, lo que favorece la propagación rápida de tendencias e información. |

### Caracterización de las comunidades detectadas

La partición por optimización de modularidad identificó **6 comunidades** con perfiles temáticos bien diferenciados:

| Comunidad | Proporción de nodos | Perfil temático principal | Intereses representativos |
| :--- | :---: | :--- | :--- |
| **Comunidad 1** | `37.9%` | Gaming y Tecnología | *Gaming, Streaming, Humor, Technology, Science, Comics* |
| **Comunidad 2** | `36.2%` | Activismo, Política y Cultura | *Activism, Books, Politics, Journalism, Movies, Technology* |
| **Comunidad 3** | `13.2%` | Entretenimiento y Cultura Pop | *TV Shows, Celebrities, Music, Comedy, Pop Culture* |
| **Comunidad 4** | `11.0%` | Deportes | *Sports, NFL, Olympics, MMA* |
| **Comunidad 5** | `1.3%` | Ciencia y Espacio | *Space, NASA, Science, Astronomy, Climate, Geek Culture* |
| **Comunidad 6** | `0.4%` | Marketing y Branding | *Marketing, Advertising, Branding, Coffee* |

### Conectividad intercomunitaria y conclusiones

Al cuantificar el número de aristas que cruzan entre distintas particiones, se observa una fuerte asimetría en la integración de los grupos:

1. **Comunidad 1 – Comunidad 2 (`5,729` aristas):** Constituyen el núcleo estructural de la red. Su solapamiento en tecnología, cultura digital y actualidad genera un puente denso entre perfiles orientados al gaming/tecnología y perfiles vinculados al activismo y los medios.
2. **Comunidad 2 – Comunidad 3 (`242` aristas):** Conexión secundaria mediada principalmente por consumo cultural compartido (cine, activismo y entretenimiento generalista).
3. **Comunidad 1 – Comunidad 4 (`219` aristas):** Intersección moderada entre audiencias de videojuegos/streaming y aficionados a disciplinas deportivas.

**Hallazgos principales:**
- La agrupación basada en intereses inferidos logra segmentar de forma coherente hábitos y preferencias diferenciadas sin requerir el grafo explícito de seguidores.
- Las comunidades mayoritarias presentan una alta cohesión interna y fuertes puentes entre temáticas adyacentes, mientras que los nichos especializados (astronomía/espacio y marketing/branding) ocupan posiciones periféricas con enlaces muy acotados.

---

## Estructura del repositorio

```text
analisis-afinidad-hobbies/
├── 1-user_hashtags_creation.ipynb      # Notebook 1: Lectura de archivos SNAP y extracción de hashtags
├── 2-analisis_redsocial_twitter.ipynb  # Notebook 2: Construcción del grafo, Louvain, métricas y gráficos
├── user_hashtags.csv                   # Dataset intermedio de hashtags y menciones por usuario
├── user_interests.csv                  # Intereses inferidos y normalizados con DeepSeek
├── RED_GEPHI.gephi                     # Proyecto de visualización de red en Gephi
├── Resultados.pdf                      # Infografía resumen con métricas, grafo y observaciones
├── LICENSE                             # Licencia CC0 1.0 Universal
└── README.md                           # Documentación técnica del repositorio
```

---

## Requisitos e instalación

### Entorno recomendado
- **Python 3.10** o superior.
- **Jupyter Notebook** o **JupyterLab**.
- **Gephi** (opcional, necesario únicamente para inspeccionar `RED_GEPHI.gephi` o el archivo `red_social.gexf` exportado desde el Notebook 2).

### Dependencias de Python

Se recomienda crear un entorno virtual e instalar los paquetes requeridos:

```bash
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate
pip install numpy pandas networkx python-louvain matplotlib jupyter
```

---

## Reproducibilidad y ejecución

1. **Clonar el repositorio:**
   ```bash
   git clone https://github.com/<usuario>/analisis-afinidad-hobbies.git
   cd analisis-afinidad-hobbies
   ```

2. **Fase 1 – Extracción de hashtags (opcional):**
   - El repositorio incluye los archivos CSV resultantes para permitir la ejecución directa del análisis.
   - Si se desea reproducir el preprocesamiento desde cero, es necesario descargar el dataset [`ego-Twitter`](https://snap.stanford.edu/data/ego-Twitter.html) de SNAP, descomprimirlo en un directorio `twitter/` y actualizar las rutas locales en `1-user_hashtags_creation.ipynb`.

3. **Fase 2 – Construcción del grafo y análisis (`2-analisis_redsocial_twitter.ipynb`):**
   - Ejecutar todas las celdas del notebook para cargar `user_interests.csv`, generar el grafo ponderado en NetworkX, detectar las comunidades mediante Louvain, calcular las métricas topológicas y exportar el grafo `red_social.gexf` junto con las figuras analíticas.

4. **Fase 3 – Exploración visual:**
   - Abrir `RED_GEPHI.gephi` (o el archivo `red_social.gexf` generado) en **Gephi** para inspeccionar la distribución espacial y los atributos de nodos y comunidades.

---

## Autores y licencia

- **Autores:** Adrià, Garofalo y Bakali — *Proyecto de Computación Social y Personalización*.
- **Licencia:** Distribuido bajo licencia **CC0 1.0 Universal (Dominio Público)**. Véase el archivo [`LICENSE`](LICENSE) para consultar los términos completos.