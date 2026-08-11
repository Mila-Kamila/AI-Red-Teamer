# Applications of AI in InfoSec

**Miniconda**

```bash
PanthackMilaU@htb[/htb]$ conda init
PanthackMilaU@htb[/htb]$ conda config --add channels defaults
PanthackMilaU@htb[/htb]$ conda config --add channels conda-forge
PanthackMilaU@htb[/htb]$ conda config --add channels nvidia # solo es necesario si estás en un PC con una GPU de Nvidia
PanthackMilaU@htb[/htb]$ conda config --add channels pytorch
PanthackMilaU@htb[/htb]$ conda config --set channel_priority strict
```

**Desactivando Base**

```bash
(base) $
PanthackMilaU@htb[/htb]$ conda config --set auto_activate_base false
```

Crear un nuevo entorno

```bash
PanthackMilaU@htb[/htb]$ conda create -n ai python=3.11
```

**Activando el Entorno**

```bash
PanthackMilaU@htb[/htb]$ conda deactivate
```

**Configuración Esencial**

```bash
PanthackMilaU@htb[/htb]$ conda install -y numpy scipy pandas scikit-learn matplotlib seaborn transformers datasets tokenizers accelerate evaluate optimum huggingface_hub nltk category_encoders
PanthackMilaU@htb[/htb]$ conda install -y pytorch torchvision torchaudio pytorch-cuda=12.4 -c pytorch -c nvidia
PanthackMilaU@htb[/htb]$ pip install requests requests_toolbelt
```

**Actualizaciones**

```bash
PanthackMilaU@htb[/htb]$ conda update --all
```

**JupyterLab**

```bash
PanthackMilaU@htb[/htb]$ conda install -y jupyter jupyterlab notebook ipykernel 
```

Inicial Jupyter 

```bash
PanthackMilaU@htb[/htb]$ jupyter lab
```

# **Bibliotecas de Python para IA**

Scikit-learn (Machine Learning clásico)

- Librería para ML tradicional, construida sobre NumPy/SciPy.
- Supervisado: regresión lineal/logística, SVM, árboles, Naive Bayes, ensambles (random forest, gradient boosting).
- No supervisado: clustering (K-Means, DBSCAN), reducción de dimensionalidad (PCA, t-SNE).
- Preprocesamiento: StandardScaler/MinMaxScaler (escalado), OneHotEncoder/LabelEncoder (categóricas), SimpleImputer (valores
faltantes).
- Flujo típico:
train_test_split(X, y) → model.fit(X_train, y_train) → model.predict(X_test)
- Evaluación: cross_val_score (validación cruzada), métricas como accuracy_score, mean_squared_error, f1_score.
- Cuándo usarlo: datos tabulares, problemas clásicos de ML, prototipado rápido.

PyTorch (Deep Learning)

- Librería de Facebook/Meta para redes neuronales y deep learning.
- Tensores: como arrays de NumPy, pero corren en GPU (.to('cuda')).
- Grafos dinámicos: se construyen en tiempo real (a diferencia de TensorFlow clásico) → más flexible para debug.
- Construcción de modelos:
    - nn.Sequential → apilar capas rápido.
    - nn.Module (clase custom con forward()) → modelos complejos.
- Entrenamiento = 4 piezas:
a. Optimizador: Adam, SGD, RMSprop.
b. Loss function: CrossEntropyLoss (multiclase), BCEWithLogitsLoss (binaria), MSELoss (regresión).
c. Loop: forward pass → loss → loss.backward() → optimizer.step().
d. Datos: Dataset + DataLoader (batches, shuffle).
- Guardar/cargar modelo: torch.save(model.state_dict(), 'model.pth') / load_state_dict().
- Cuándo usarlo: imágenes, texto, audio, redes neuronales complejas, cuando necesitas GPU.

Regla mental rápida

Scikit-learn = ML "tradicional" (tabular, algoritmos clásicos) → PyTorch = Deep Learning (redes neuronales, GPU, datos no
estructurados)

# Conjuntos de datos (Datasets)

Tipos de datos:

- Tabulares: filas/columnas (hojas de cálculo, BD).
- Imágenes: matrices numéricas de píxeles.
- Texto: no estructurado (frases, documentos).
- Series temporales: secuenciales, con patrón temporal.

Por qué importa la calidad: precisión del modelo, generalización (evita overfitting), eficiencia (menos tiempo/cómputo), fiabilidad (crítico en salud/finanzas).

Atributos de un buen dataset

| Atributo  | Qué significa  |
| --- | --- |
|  Relevancia  |  Datos pertinentes al problema  |
|  Completitud  |  Pocos valores faltantes  |
| Consistencia |  Formato/estructura uniforme (ej: fechas)  |
| Calidad  | Sin errores de captura  |
| Representatividad | Refleja bien la población real (evita sesgo)  |
|  Balance  |  Clases equilibradas (sino: oversampling/undersampling)  |
|  Tamaño |  Suficiente para aprender, sin ser inmanejable |

Caso práctico: demo_dataset.csv (logs de red)

Columnas: log_id, source_ip, destination_port, protocol, bytes_transferred, threat_level (0=normal, 1=baja amenaza, 2=alta
amenaza).

Problemas típicos a esperar:

- Mezcla de datos numéricos y categóricos.
- Valores faltantes / entradas inválidas.
- Strings no numéricos en columnas numéricas.
- Valores desconocidos en threat_level (?, -1) → estandarizar.

Flujo con pandas

import pandas as pd
data = pd.read_csv("./demo_dataset.csv")

data.head()          # vista rápida de filas
[data.info](http://data.info/)()          # tipos de dato + nulos por columna
data.isnull().sum()  # conteo de valores faltantes por columna

Regla mental rápida

Explorar antes de limpiar: head() → info() → isnull().sum(), en ese orden, antes de tocar el preprocesamiento.

# Preprocesamiento de datos

4 técnicas clave:

- Limpieza: valores faltantes, duplicados, ruido.
- Transformación: normalización, codificación, escalado, reducción.
- Integración: combinar datos de varias fuentes.
- Formateo: convertir tipos, remodelar estructuras.

Detectar valores no válidos

Antes de limpiar, hay que encontrar los datos malos. Patrón general: escribir una función is_valid_X() y filtrar el DataFrame con ella.

- IPs: validar con regex (^(\d{1,3}\.){3}\d{1,3}$ + rangos 0-255).
- Puertos: deben ser numéricos y estar entre 0-65535.
- Protocolo: debe estar en una lista de protocolos conocidos (TCP, UDP, HTTP, etc.).
- Bytes transferidos: numérico y no negativo.
- Threat level: numérico y dentro del rango esperado (0-2).

invalid_ips = data[~data['source_ip'].astype(str).apply(is_valid_ip)]

Dos formas de manejar lo inválido

Opción 1 — Eliminar filas malas
data = data.drop(invalid_ips.index, errors='ignore')
Simple y seguro, pero pierdes datos (en el ejemplo, de un dataset grande quedaron solo 77 filas limpias). Úsalo cuando la precisión importa más que la cantidad.

Opción 2 — Imputar (rellenar en vez de borrar)
Se prefiere cuando quieres conservar la mayor cantidad de información posible. El proceso tiene 3 pasos:

1. Estandarizar lo inválido a NaN: reemplaza strings raros (MISSING_IP, ?, NON_NUMERIC, etc.) por np.nan con df.replace(...), y usa pd.to_numeric(errors='coerce') para forzar columnas numéricas.
2. Imputar valores:
- Numéricas → SimpleImputer(strategy='median') (mediana es más robusta que la media ante outliers).
- Categóricas → SimpleImputer(strategy='most_frequent').
- Método avanzado → KNNImputer (usa relaciones entre columnas, no solo un valor fijo).
3. Aplicar reglas de dominio (sentido común del problema):
- IP faltante → 0.0.0.0 como default.
- Protocolo inválido → reemplazar por el más frecuente (moda).
- Puertos fuera de rango → recortar con .clip(lower=0, upper=65535).

Finaliza siempre revisando con df.describe(include='all') para confirmar que las distribuciones tienen sentido.

Regla mental rápida

Eliminar = rápido y limpio pero pierdes datos. Imputar = conservas datos pero necesitas 3 pasos: estandarizar a NaN → imputar → aplicar reglas de dominio.

# Transformación de datos

Mejora cómo se representan y distribuyen las características para que el modelo las aproveche mejor: convierte categorías a números y corrige distribuciones sesgadas.

Codificación de variables categóricas

Convertir texto/categorías a números. Opciones según el caso:

- OneHotEncoder → crea una columna binaria (0/1) por cada categoría. No inventa un orden falso. Ideal cuando hay pocas categorías.
- LabelEncoder → asigna un número entero a cada categoría (0, 1, 2...). Cuidado: esto puede hacer que el modelo interprete un orden que no existe.
- HashingEncoder / frecuencia → para cuando hay MUCHAS categorías distintas (alta cardinalidad) y no quieres que exploten las columnas.

Ejemplo mental: color con valores red/green/blue → OneHot crea color_red, color_green, color_blue, y solo una tiene 1 por fila.

from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(handle_unknown='ignore', sparse_output=False)
encoded = encoder.fit_transform(df[['protocol']])
encoded_df = pd.DataFrame(encoded, columns=encoder.get_feature_names_out(['protocol']))
df = pd.concat([df.drop('protocol', axis=1), encoded_df], axis=1)

Datos sesgados (skewed)

Cuando la mayoría de los valores están agrupados y unos pocos extremos "estiran" la distribución (ej: bytes transferidos con algunos valores gigantes). Esto afecta a modelos sensibles a outliers.

Solución típica: transformación logarítmica → comprime los valores grandes mucho más que los pequeños, dejando una distribución más pareja.

df["bytes_transferred"] = np.log1p(df["bytes_transferred"])  # log(x+1), evita log(0)

Resultado: el modelo deja de estar dominado por los valores extremos y generaliza mejor.

División de datos (train / validation / test)

Nunca entrenas y evalúas con los mismos datos. Se divide en 3 partes:

- Entrenamiento (~60-80%): el modelo aprende de aquí.
- Validación (~10-20%): para ajustar hiperparámetros y comparar modelos.
- Prueba (~10-20%): se usa solo al final, datos que el modelo nunca vio.

Truco del código: se hace en 2 pasos con train_test_split. Primero separas test (20%), luego vuelves a dividir el 80% restante para sacar la validación (25% de ese 80% = 20% del total).

from sklearn.model_selection import train_test_split

X = df.drop("threat_level", axis=1)
y = df["threat_level"]

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=1337)
X_train, X_val, y_train, y_val = train_test_split(X_train, y_train, test_size=0.25, random_state=1337)

Flujo final: entrenar con train → ajustar con val → evaluar de verdad solo una vez con test.

Regla mental rápida

OneHot para categorías sin orden, log1p para corregir sesgo en números, train/val/test para que la evaluación final sea honesta (nunca tocar test hasta el final).

# Métricas para evaluar un modelo

Miden qué tan bien predice el modelo comparando predicciones vs. etiquetas reales. No basta con una sola métrica — cada una muestra un ángulo distinto.

Exactitud (Accuracy)

% de predicciones correctas en total.
(VP + VN) / total

Trampa importante: engañosa con clases desbalanceadas. Ejemplo: si 99% de los emails son legítimos, un modelo que SIEMPRE dice "no es spam" tiene 99% de exactitud pero nunca detecta spam. Alta exactitud no
significa modelo útil.

Precisión (Precision)

De todo lo que el modelo marcó como positivo, ¿cuánto acertó?
VP / (VP + FP)

Alta precisión = pocas falsas alarmas. Pero si el modelo casi nunca marca nada como positivo, puede tener precisión alta y aun así dejar pasar mucho spam real (no detecta todo).

Exhaustividad (Recall)

De todos los positivos reales, ¿cuántos detectó el modelo?
VP / (VP + FN)

Alto recall = casi no se le escapa nada. Pero si es demasiado agresivo, puede marcar de más (muchos falsos positivos) y saturar al usuario con falsas alarmas.

Precisión vs Recall = tensión constante: mejorar uno suele empeorar el otro.

F1-score

Balance entre precisión y recall (media armónica, no promedio simple).
2 * (precisión * recall) / (precisión + recall)

Se usa cuando quieres un solo número que no favorezca a ninguna de las dos, útil sobre todo con clases desbalanceadas.

Otras métricas a conocer

- Especificidad: qué tan bien detecta los negativos (lo opuesto al recall).
- AUC (ROC): qué tan bien distingue el modelo entre clases en distintos umbrales.
- Matthews Correlation Coefficient (MCC): mejor que accuracy cuando hay mucho desbalance.
- Matriz de confusión: tabla resumen de VP, VN, FP, FN — la base de donde salen todas las demás métricas.

Cómo interpretar las métricas en contexto

No mires el número solo, pregúntate:

- ¿Se mantiene consistente en distintos subgrupos de datos?
- ¿El dataset refleja la realidad (incluye el desbalance real)?
- ¿Cuál es más costoso, un falso positivo o un falso negativo?

Regla práctica según el caso de uso:

- Detección de amenazas/seguridad → prioriza recall (mejor una falsa alarma que dejar pasar un ataque real).
- Recursos limitados / evitar ruido → prioriza precisión (evitar perseguir falsas alarmas).

Regla mental rápida

Accuracy = visión general (cuidado si hay desbalance) → Precision = "cuando digo sí, ¿acierto?" → Recall = "¿detecto todos los sí reales?" → F1 = balance entre ambas.

# Clasificación de spam (Naive Bayes)

Detectar spam es clave para seguridad (phishing) y usabilidad de sistemas de mensajería. Uno de los enfoques clásicos y efectivos es Naive Bayes, basado en probabilidad.

Teorema de Bayes (la base)

P(A|B) = (P(B|A) * P(A)) / P(B)
Traducido a spam: "¿cuál es la probabilidad de que sea spam, dado que veo estas palabras/características?"

- P(Spam|Features) → lo que queremos saber (posterior).
- P(Features|Spam) → verosimilitud (qué tan común es ver esas palabras en spam).
- P(Spam) → prior, probabilidad base de que cualquier correo sea spam.
- P(Features) → probabilidad total de ver esas características (normaliza el resultado).

La parte "ingenua" (Naive)

Asume que cada característica es independiente de las demás dado que ya sabes la clase. Esto simplifica todo a una multiplicación simple:
P(Features|Spam) = P(feature1|Spam) * P(feature2|Spam) * ...
No es 100% realista (las palabras suelen estar correlacionadas), pero funciona muy bien en la práctica y es rápido de calcular.

Cómo se decide

Se calcula P(Spam|Features) y P(Not Spam|Features) → gana la clase con mayor probabilidad.

Ejemplo numérico (la lógica, no memorizar los números)

Con dos características F1 y F2:

1. Multiplicas las probabilidades individuales de cada característica dado Spam → obtienes P(F1,F2|Spam).
2. Haces lo mismo para Not Spam.
3. Combinas con el prior de cada clase (P(Spam), P(Not Spam)).
4. Divides entre la probabilidad total (P(F1,F2)) para normalizar.
5. Comparas: la clase con el valor más alto es la predicción final.

En el ejemplo del texto: P(Spam|F1,F2) ≈ 0.588 vs P(Not Spam|F1,F2) ≈ 0.412 → se clasifica como spam, porque 0.588 > 0.412.

Regla mental rápida

Naive Bayes = multiplicar probabilidades de cada palabra/característica por separado, asumiendo que no se afectan entre sí, y quedarte con la clase (spam / no spam) que dé el número más alto.

# El conjunto de datos de spam (SMS Spam Collection)

Qué es: dataset de 5,574 SMS etiquetados como ham (legítimo) o spam, hecho por investigadores de Brasil y España para resolver un problema específico: la mayoría de recursos de detección de spam existían
para email, no para SMS.

- ham = mensajes válidos (contactos, boletines, etc.)
- spam = mensajes no solicitados/dañinos

Descargar y extraer el dataset

import requests, zipfile, io

url = "[https://archive.ics.uci.edu/static/public/228/sms+spam+collection.zip](https://archive.ics.uci.edu/static/public/228/sms+spam+collection.zip)"
response = requests.get(url)  # status_code == 200 → OK

with zipfile.ZipFile(io.BytesIO(response.content)) as z:
z.extractall("sms_spam_collection")

- response.content = bytes crudos del zip descargado.
- io.BytesIO los convierte en algo que zipfile puede leer sin guardar el archivo primero.
- os.listdir("sms_spam_collection") sirve para confirmar qué se extrajo.

Cargar el dataset

El archivo es TSV (separado por tabulaciones), sin fila de encabezado:
df = pd.read_csv(
"sms_spam_collection/SMSSpamCollection",
sep="\t",
header=None,
names=["label", "message"],
)

- sep="\t" → indica que el separador es tabulación, no coma.
- header=None + names=[...] → como el archivo no trae nombres de columna, se los pones tú manualmente.

Inspección básica (siempre igual)

df.head()      # vista rápida de filas
df.describe()  # resumen estadístico (útil para ver distribución de labels)
[df.info](http://df.info/)()      # tipos de dato + conteo de no-nulos

Revisar calidad de los datos

Valores faltantes:
df.isnull().sum()

Duplicados (pueden sesgar el análisis si un mismo mensaje se repite mucho):
df.duplicated().sum()   # cuántos hay
df = df.drop_duplicates()  # eliminarlos

Regla mental rápida

Flujo estándar para cualquier dataset descargado: descargar → extraer → cargar con el separador correcto → head/info/describe → revisar nulos → revisar y eliminar duplicados. Este mismo patrón se repite
siempre, cambia solo el dataset.

# Preprocesamiento del dataset de spam (texto)

Antes de entrenar el clasificador, hay que limpiar y normalizar el texto de los SMS. Se usa nltk para tokenizar, quitar palabras vacías y aplicar stemming. Pasos en orden:

1. Descargar recursos de NLTK

nltk.download("punkt")       # para tokenizar
nltk.download("punkt_tab")
nltk.download("stopwords")   # para quitar palabras vacías

1. Minúsculas

Evita que "Free" y "free" se traten como palabras distintas.
df["message"] = df["message"].str.lower()

1. Quitar puntuación y números (pero no todo)

Se limpia el texto, pero se conservan $y ! porque aportan señal real de spam (dinero, urgencia/énfasis).
  df["message"] = df["message"].apply(lambda x: re.sub(r"[^a-z\s$!]", "", x))

1. Tokenizar

Divide cada mensaje en una lista de palabras individuales (tokens). Necesario antes de poder filtrar o transformar palabra por palabra.
df["message"] = df["message"].apply(word_tokenize)

1. Eliminar stop words

Quita palabras muy comunes sin significado propio (and, the, is...) para que el modelo se enfoque en las palabras que sí distinguen spam de ham.
stop_words = set(stopwords.words("english"))
df["message"] = df["message"].apply(lambda x: [w for w in x if w not in stop_words])

1. Stemming (derivación)

Reduce las palabras a su raíz (running → run). Junta variantes de una misma palabra para que el modelo no las trate como conceptos distintos, reduciendo el vocabulario.
stemmer = PorterStemmer()
df["message"] = df["message"].apply(lambda x: [stemmer.stem(w) for w in x])

1. Volver a unir los tokens en texto

Muchos algoritmos (como TF-IDF) esperan strings, no listas de palabras. Se re-arma el mensaje como una sola cadena limpia.
df["message"] = df["message"].apply(lambda x: " ".join(x))

Regla mental rápida

Orden fijo del pipeline de texto: minúsculas → limpiar símbolos (conservando señales útiles como $ y !) → tokenizar → quitar stop words → stemming → volver a unir en string. Al final tienes texto limpio y
normalizado, listo para vectorizar (TF-IDF) y entrenar el modelo.

# Extracción de características (texto → números)

Los modelos no entienden texto crudo, necesitan vectores numéricos. La técnica clásica es bag-of-words (bolsa de palabras).

Idea central: Bag-of-words

- Construye un vocabulario con todos los términos únicos del dataset.
- Cada mensaje se convierte en un vector donde cada posición = cuántas veces aparece ese término.
- Limitación: no preserva el orden real de las palabras, solo cuenta frecuencias.

Unigramas vs Bigramas

- Unigramas = palabras sueltas ("free", "prize").
- Bigramas = pares de palabras consecutivas ("free prize").

Agregar bigramas da un poco de contexto/orden que los unigramas solos pierden. Ejemplo: "free prize" (spam típico) se distingue de solo "free" suelto en cualquier frase normal. Pero solo captura orden local
(definido por ngram_range), no la oración completa.

CountVectorizer (scikit-learn)

Implementa bag-of-words automáticamente: tokeniza, arma vocabulario y convierte cada mensaje en un vector de conteos.

from sklearn.feature_extraction.text import CountVectorizer

vectorizer = CountVectorizer(min_df=1, max_df=0.9, ngram_range=(1, 2))
X = vectorizer.fit_transform(df["message"])
y = df["label"].apply(lambda x: 1 if x == "spam" else 0)

Parámetros clave:

- min_df → un término debe aparecer al menos en N documentos para contar (filtra términos rarísimos).
- max_df=0.9 → descarta palabras que aparecen en más del 90% de los mensajes (demasiado comunes, no discriminan — ej: "the").
- ngram_range=(1, 2) → incluye unigramas Y bigramas en el vocabulario.

Las 3 etapas internas de CountVectorizer

1. Tokenización: separa el texto en unigramas/bigramas según ngram_range.
2. Construcción del vocabulario: filtra usando min_df y max_df para quedarse con términos ni muy raros ni muy comunes.
3. Vectorización: convierte cada mensaje en un vector de conteos, uno por término del vocabulario.

Resultado final: cada mensaje SMS se convierte en una fila numérica dentro de una matriz X, lista para entrenar un clasificador (ej: Naive Bayes).

Regla mental rápida

CountVectorizer = cuenta palabras (y pares de palabras) por mensaje, filtrando lo demasiado raro (min_df) y lo demasiado común (max_df). Más ngram_range = más contexto, pero también más columnas (más dimensiones).

# Entrenamiento y evaluación (Detección de spam)

Entrenamiento

Modelo elegido: Multinomial Naive Bayes, ideal para texto porque maneja bien muchas características (palabras) y datos dispersos (sparse).

Pipeline: encadena vectorización + clasificador en un solo objeto, así siempre se aplica la misma transformación antes de predecir (evita errores de "olvidé vectorizar igual que en entrenamiento").
pipeline = Pipeline([
("vectorizer", vectorizer),
("classifier", MultinomialNB())
])

Ajuste de hiperparámetros con GridSearchCV: prueba distintos valores de alpha (suavizado — evita que una palabra nueva/no vista haga que la probabilidad sea 0) y se queda con el mejor usando validación
cruzada.
param_grid = {"classifier__alpha": [0.01, 0.1, 0.15, 0.2, 0.25, 0.5, 0.75, 1.0]}

grid_search = GridSearchCV(pipeline, param_grid, cv=5, scoring="f1")
grid_search.fit(df["message"], y)

best_model = grid_search.best_estimator_

- cv=5 → validación cruzada de 5 particiones.
- scoring="f1" → elige el mejor alpha según F1-score (balance precisión/recall), no accuracy.

Evaluación con mensajes nuevos

Regla de oro: los mensajes nuevos deben pasar por el mismo preprocesamiento que los datos de entrenamiento (minúsculas → limpiar símbolos → tokenizar → quitar stop words → stemming).
def preprocess_message(message):
message = message.lower()
message = re.sub(r"[^a-z\s$!]", "", message)
tokens = word_tokenize(message)
tokens = [w for w in tokens if w not in stop_words]
tokens = [stemmer.stem(w) for w in tokens]
return " ".join(tokens)

Luego se vectoriza con el mismo vectorizer ya entrenado (no uno nuevo):
X_new = best_model.named_steps["vectorizer"].transform(processed_messages)

Y se predice con el clasificador, obteniendo tanto la etiqueta como la probabilidad (confianza):
predictions = best_model.named_steps["classifier"].predict(X_new)
prediction_probabilities = best_model.named_steps["classifier"].predict_proba(X_new)

predict_proba da dos números por mensaje: probabilidad de ser ham y probabilidad de ser spam — útil para ver qué tan "seguro" está el modelo, no solo la decisión final.

Guardar el modelo con joblib

Entrenar cuesta tiempo/cómputo — no quieres reentrenar cada vez que reinicias la app. joblib serializa el modelo completo (pipeline + parámetros aprendidos) a un archivo binario.

Guardar:
import joblib
joblib.dump(best_model, 'spam_detection_model.joblib')

Cargar y usar después:
loaded_model = joblib.load('spam_detection_model.joblib')
predictions = loaded_model.predict(new_data_processed)  # datos ya preprocesados igual que en train

Regla mental rápida

Flujo completo: Pipeline (vectorizer + modelo) → GridSearchCV para encontrar mejor alpha → evaluar con mensajes nuevos preprocesados IGUAL que en training → guardar con joblib para no reentrenar cada vez.

# Evaluación del modelo (Detección de spam)

Esta parte es específica de HTB (Hack The Box) / Playground VM — no es teoría general, es el paso de "entrega" del ejercicio.

Qué hace este paso

Subes tu modelo entrenado (spam_detection_model.joblib) a un portal web que corre en la Playground VM. El portal evalúa el modelo contra criterios de rendimiento (accuracy, F1, etc.) y, si pasa, te da una
flag como prueba de éxito.

Subir el modelo desde Jupyter (VM ya corriendo)

import requests, json

url = "[http://localhost:8000/api/upload](http://localhost:8000/api/upload)"
model_file_path = "spam_detection_model.joblib"

with open(model_file_path, "rb") as model_file:
files = {"model": model_file}
response = requests.post(url, files=files)

print(json.dumps(response.json(), indent=4))

- Envía el archivo como multipart/form-data (files=...), no como JSON.
- La respuesta trae el resultado de la evaluación (y la flag si aprueba).

Si trabajas desde tu propia máquina (no en la VM)

1. Conectar la VPN de HTB a la VM remota.
2. Iniciar la Playground VM.
3. Ir a http://<VM-IP>:8000/ en el navegador y subir el modelo manualmente ahí (en vez de usar localhost).

Regla mental rápida

localhost:8000 si ya estás dentro de la VM (Jupyter) → http://<VM-IP>:8000/ desde tu propia máquina vía VPN. El resultado final es la flag, que confirma que el modelo cumple el umbral de rendimiento pedido

# Code

1. Instalar/importar NLTK y descargar recursos
import nltk
nltk.download("punkt")
nltk.download("punkt_tab")
nltk.download("stopwords")
2. Descargar y extraer el dataset
import requests, zipfile, io, os

url = "[https://archive.ics.uci.edu/static/public/228/sms+spam+](https://archive.ics.uci.edu/static/public/228/sms+spam+)
collection.zip"
response = requests.get(url)
with zipfile.ZipFile(io.BytesIO(response.content)) as z:
z.extractall("sms_spam_collection")

print(os.listdir("sms_spam_collection"))

1. Cargar el dataset
import pandas as pd

df = pd.read_csv(
"sms_spam_collection/SMSSpamCollection",
sep="\t", header=None, names=["label", "message"]
)
df = df.drop_duplicates()
print(df.shape)

1. Preprocesar el texto
import re
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
from nltk.stem import PorterStemmer

stop_words = set(stopwords.words("english"))
stemmer = PorterStemmer()

df["message"] = df["message"].str.lower()
df["message"] = df["message"].apply(lambda x:
re.sub(r"[^a-z\s$!]", "", x))
df["message"] = df["message"].apply(word_tokenize)
df["message"] = df["message"].apply(lambda x: [w for w in x if
w not in stop_words])
df["message"] = df["message"].apply(lambda x: [stemmer.stem(w)
for w in x])
df["message"] = df["message"].apply(lambda x: " ".join(x))

1. Entrenar el modelo (pipeline + GridSearch)
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline
from sklearn.model_selection import GridSearchCV

y = df["label"].apply(lambda x: 1 if x == "spam" else 0)

vectorizer = CountVectorizer(min_df=1, max_df=0.9,
ngram_range=(1, 2))

pipeline = Pipeline([
("vectorizer", vectorizer),
("classifier", MultinomialNB())
])

param_grid = {"classifier__alpha": [0.01, 0.1, 0.15, 0.2,
0.25, 0.5, 0.75, 1.0]}

grid_search = GridSearchCV(pipeline, param_grid, cv=5,
scoring="f1")
grid_search.fit(df["message"], y)

best_model = grid_search.best_estimator_
print("Mejores parámetros:", grid_search.best_params_)

1. Guardar el modelo
import joblib

joblib.dump(best_model, "spam_detection_model.joblib")

1. Subir el modelo para obtener la flag

Primero asegúrate de tener la Playground VM corriendo (botón
al final de la página del módulo), y verifica que puedes
acceder a http://<VM-IP>:8000/.

import requests, json

url = "[http://localhost:8000/api/upload](http://localhost:8000/api/upload)"  # o
http://<IP-VM>:8000/api/upload si no es local

with open("spam_detection_model.joblib", "rb") as model_file:
files = {"model": model_file}
response = requests.post(url, files=files)

print(json.dumps(response.json(), indent=4))

La respuesta JSON debería incluir la flag si el modelo cumple
el umbral de rendimiento requerido.

1. Descargar y extraer el dataset
import requests, zipfile, io

url = "[https://academy.hackthebox.com/storage/modules/292/KDD_](https://academy.hackthebox.com/storage/modules/292/KDD_)
dataset.zip"
response = requests.get(url)
z = zipfile.ZipFile(io.BytesIO(response.content))
z.extractall('.')

1. Cargar el dataset y definir columnas
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, precision_score,
recall_score, f1_score, confusion_matrix,
classification_report
import seaborn as sns
import matplotlib.pyplot as plt

file_path = r'KDD+.txt'

columns = [
'duration', 'protocol_type', 'service', 'flag',
'src_bytes', 'dst_bytes',
'land', 'wrong_fragment', 'urgent', 'hot',
'num_failed_logins', 'logged_in',
'num_compromised', 'root_shell', 'su_attempted',
'num_root', 'num_file_creations',
'num_shells', 'num_access_files', 'num_outbound_cmds',
'is_host_login', 'is_guest_login',
'count', 'srv_count', 'serror_rate', 'srv_serror_rate',
'rerror_rate', 'srv_rerror_rate',
'same_srv_rate', 'diff_srv_rate', 'srv_diff_host_rate',
'dst_host_count', 'dst_host_srv_count',
'dst_host_same_srv_rate', 'dst_host_diff_srv_rate',
'dst_host_same_src_port_rate',
'dst_host_srv_diff_host_rate', 'dst_host_serror_rate',
'dst_host_srv_serror_rate',
'dst_host_rerror_rate', 'dst_host_srv_rerror_rate',
'attack', 'level'
]

df = pd.read_csv(file_path, names=columns)
print(df.head())

1. Etiquetas binaria y multiclase
df['attack_flag'] = df['attack'].apply(lambda a: 0 if a ==
'normal' else 1)

dos_attacks = ['apache2', 'back', 'land', 'neptune',
'mailbomb', 'pod',
'processtable', 'smurf', 'teardrop',
'udpstorm', 'worm']
probe_attacks = ['ipsweep', 'mscan', 'nmap', 'portsweep',
'saint', 'satan']
privilege_attacks = ['buffer_overflow', 'loadmdoule', 'perl',
'ps',
'rootkit', 'sqlattack', 'xterm']
access_attacks = ['ftp_write', 'guess_passwd', 'http_tunnel',
'imap',
'multihop', 'named', 'phf', 'sendmail',
'snmpgetattack',
'snmpguess', 'spy', 'warezclient',
'warezmaster',
'xclock', 'xsnoop']

def map_attack(attack):
if attack in dos_attacks:
return 1
elif attack in probe_attacks:
return 2
elif attack in privilege_attacks:
return 3
elif attack in access_attacks:
return 4
else:
return 0

df['attack_map'] = df['attack'].apply(map_attack)

1. Codificar variables y armar el set de entrenamiento
features_to_encode = ['protocol_type', 'service']
encoded = pd.get_dummies(df[features_to_encode])

numeric_features = [
'duration', 'src_bytes', 'dst_bytes', 'wrong_fragment',
'urgent', 'hot',
'num_failed_logins', 'num_compromised', 'root_shell',
'su_attempted',
'num_root', 'num_file_creations', 'num_shells',
'num_access_files',
'num_outbound_cmds', 'count', 'srv_count', 'serror_rate',
'srv_serror_rate', 'rerror_rate', 'srv_rerror_rate',
'same_srv_rate',
'diff_srv_rate', 'srv_diff_host_rate', 'dst_host_count',
'dst_host_srv_count',
'dst_host_same_srv_rate', 'dst_host_diff_srv_rate',
'dst_host_same_src_port_rate',
'dst_host_srv_diff_host_rate',
'dst_host_serror_rate', 'dst_host_srv_serror_rate',
'dst_host_rerror_rate',
'dst_host_srv_rerror_rate'
]

train_set = encoded.join(df[numeric_features])
multi_y = df['attack_map']

train_X, test_X, train_y, test_y = train_test_split(train_set,
multi_y, test_size=0.2, random_state=1337)
multi_train_X, multi_val_X, multi_train_y, multi_val_y =
train_test_split(train_X, train_y, test_size=0.3,
random_state=1337)

1. Entrenar el modelo
rf_model_multi = RandomForestClassifier(random_state=1337)
rf_model_multi.fit(multi_train_X, multi_train_y)
2. Evaluar (opcional, para ver métricas)
multi_predictions = rf_model_multi.predict(multi_val_X)
print("Accuracy:", accuracy_score(multi_val_y,
multi_predictions))
print("F1:", f1_score(multi_val_y, multi_predictions,
average='weighted'))
3. Guardar el modelo
import joblib

model_filename = 'network_anomaly_detection_model.joblib'
joblib.dump(rf_model_multi, model_filename)

1. Subir el modelo para obtener la flag

Igual que antes: usa la IP de tu Playground VM (10.129.69.43 o
la que te muestre ahora), pero con el puerto 8001 en vez de
8000, ya que este módulo usa un endpoint distinto.

import requests, json

url = "[http://10.129.69.43:8001/api/upload](http://10.129.69.43:8001/api/upload)"

with open("network_anomaly_detection_model.joblib", "rb") as
model_file:
files = {"model": model_file}
response = requests.post(url, files=files)

print(json.dumps(response.json(), indent=4))

O simplemente ve a [http://10.129.69.43:8001/](http://10.129.69.43:8001/) en el navegador y
sube el archivo .joblib con el botón "Browse" como hiciste
antes.

1. Instalar dependencias (en la terminal, con el entorno activado)
conda install -y pytorch torchvision -c pytorch
pip install split-folders seaborn matplotlib requests
2. Descargar y descomprimir el dataset (en terminal)
wget [https://www.kaggle.com/api/v1/datasets/download/ikrambenabd/malimg-original](https://www.kaggle.com/api/v1/datasets/download/ikrambenabd/malimg-original) -O malimg.zip
unzip malimg.zip
3. Ver distribución de clases (celda, opcional)
import os
import matplotlib.pyplot as plt
import seaborn as sns

DATA_BASE_PATH = "./malimg_paper_dataset_imgs/"

dist = {}
for mlw_class in os.listdir(DATA_BASE_PATH):
mlw_dir = os.path.join(DATA_BASE_PATH, mlw_class)
dist[mlw_class] = len(os.listdir(mlw_dir))

print(dist)

1. Dividir en train/val/test
import splitfolders

DATA_BASE_PATH = "./malimg_paper_dataset_imgs/"
TARGET_BASE_PATH = "./newdata/"

TRAINING_RATIO = 0.8
TEST_RATIO = 1 - TRAINING_RATIO

splitfolders.ratio(input=DATA_BASE_PATH, output=TARGET_BASE_PATH, ratio=(TRAINING_RATIO, 0, TEST_RATIO))

1. Funciones de carga de datos
from torchvision import transforms
from torch.utils.data import DataLoader
from torchvision.datasets import ImageFolder
import os

def load_datasets(base_path, train_batch_size, test_batch_size):
transform = transforms.Compose([
transforms.Resize((75, 75)),
transforms.ToTensor(),
transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
])

```
  train_dataset = ImageFolder(root=os.path.join(base_path, "train"), transform=transform)
  test_dataset = ImageFolder(root=os.path.join(base_path, "test"), transform=transform)

  train_loader = DataLoader(train_dataset, batch_size=train_batch_size, shuffle=True, num_workers=2)
  test_loader = DataLoader(test_dataset, batch_size=test_batch_size, shuffle=False, num_workers=2)

  n_classes = len(train_dataset.classes)
  return train_loader, test_loader, n_classes
```

1. Definir el modelo
import torch.nn as nn
import torchvision.models as models

HIDDEN_LAYER_SIZE = 1000

class MalwareClassifier(nn.Module):
def **init**(self, n_classes):
super(MalwareClassifier, self).**init**()
self.resnet = models.resnet50(weights='DEFAULT')

```
      for param in self.resnet.parameters():
          param.requires_grad = False

      num_features = self.resnet.fc.in_features
      self.resnet.fc = nn.Sequential(
          nn.Linear(num_features, HIDDEN_LAYER_SIZE),
          nn.ReLU(),
          nn.Linear(HIDDEN_LAYER_SIZE, n_classes)
      )

  def forward(self, x):
      return self.resnet(x)
```

1. Funciones de entrenamiento, evaluación y guardado
import torch
import time

def compute_accuracy(n_correct, n_total):
return round(100 * n_correct / n_total, 2)

def train(model, train_loader, n_epochs, verbose=False):
model.train()
criterion = torch.nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters())

```
  training_data = {"accuracy": [], "loss": []}

  for epoch in range(n_epochs):
      running_loss = 0
      n_total = 0
      n_correct = 0
      checkpoint = time.time() * 1000

      for inputs, labels in train_loader:
          optimizer.zero_grad()
          outputs = model(inputs)
          loss = criterion(outputs, labels)
          loss.backward()
          optimizer.step()

          _, predicted = outputs.max(1)
          n_total += labels.size(0)
          n_correct += predicted.eq(labels).sum().item()
          running_loss += loss.item()

      epoch_loss = running_loss / len(train_loader)
      epoch_duration = int(time.time() * 1000 - checkpoint)
      epoch_accuracy = compute_accuracy(n_correct, n_total)

      training_data["accuracy"].append(epoch_accuracy)
      training_data["loss"].append(epoch_loss)

      if verbose:
          print(f"[i] Epoch {epoch+1} of {n_epochs}: Acc: {epoch_accuracy:.2f}% Loss: {epoch_loss:.4f} (Took {epoch_duration}
```

ms).")

```
  return training_data
```

def save_model(model, path):
model_scripted = torch.jit.script(model)
model_scripted.save(path)

def predict(model, test_data):
model.eval()
with torch.no_grad():
output = model(test_data)
_, predicted = torch.max(output.data, 1)
return predicted

def evaluate(model, test_loader):
model.eval()
n_correct = 0
n_total = 0
with torch.no_grad():
for data, target in test_loader:
predicted = predict(model, data)
n_total += target.size(0)
n_correct += (predicted == target).sum().item()
return compute_accuracy(n_correct, n_total)

1. Entrenar (celda principal — puede tardar, usa al menos 3 épocas)
DATA_PATH = "./newdata/"
N_EPOCHS = 10
TRAINING_BATCH_SIZE = 512
TEST_BATCH_SIZE = 1024
MODEL_FILE = "malware_classifier.pth"

train_loader, test_loader, n_classes = load_datasets(DATA_PATH, TRAINING_BATCH_SIZE, TEST_BATCH_SIZE)

model = MalwareClassifier(n_classes)

print("[i] Starting Training...")
training_information = train(model, train_loader, N_EPOCHS, verbose=True)

save_model(model, MODEL_FILE)

accuracy = evaluate(model, test_loader)
print(f"[i] Inference accuracy: {accuracy}%.")

1. Subir el modelo para obtener la flag

Puerto 8002 para este módulo:
import requests, json

url = "[http://10.129.69.43:8002/api/upload](http://10.129.69.43:8002/api/upload)"

with open("malware_classifier.pth", "rb") as model_file:
files = {"model": model_file}
response = requests.post(url, files=files)

print(json.dumps(response.json(), indent=4))

O súbelo en [http://10.129.205.188:8002/](http://10.129.205.188:8002/) desde el navegador con el botón Browse.

Nota: entrenar tarda bastante (hasta ~10 min por época en la Playground VM); con 3 épocas ya debería alcanzar la precisión
requerida, pero puedes dejarlo en 10 si tienes tiempo.