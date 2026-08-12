# Introducción al Red Teaming de sistemas basados en ML

## **Red Teaming de sistemas basados en ML**

Los sistemas basados en ML se enfrentan a vulnerabilidades únicas porque dependen de grandes conjuntos de datos, inferencia estadística y arquitecturas de modelos complejas. 

Además, los sistemas basados en ML se componen de varios componentes que interactúan entre sí. A menudo, las vulnerabilidades de seguridad surgen en estos puntos de interacción. Por ello, es beneficioso incluir todos estos componentes en la evaluación de seguridad. 

# **Atacando sistemas basados en ML (Top 10 de ML de OWASP)**

Al igual que para las [Aplicaciones web](https://owasp.org/www-project-top-ten/), las [API web](https://owasp.org/www-project-api-security/) y las [Aplicaciones móviles](https://owasp.org/www-project-mobile-top-10/), OWASP ha publicado una lista Top 10 de riesgos de seguridad relacionados con el despliegue y la gestión de sistemas basados en ML

| ID | Descripción |
| --- | --- |
| ML01 | `Ataque de manipulación de entradas(Input Manipulation Attack)`: Los atacantes modifican los datos de entrada para provocar salidas incorrectas o maliciosas del modelo. 

Por ejemplo, considera un coche autónomo que utiliza un sistema basado en ML para la clasificación de imágenes de señales de tráfico para detectar el límite de velocidad actual, señales de alto y otra información relevante. En un ataque de manipulación de entradas, un atacante podría añadir pequeñas perturbaciones, como manchas de suciedad colocadas particularmente, pequeñas pegatinas o grafitis, a las señales de tráfico.  |
| ML02 | `Ataque de envenenamiento de datos(Data Poisoning Attack)`: Los atacantes inyectan datos maliciosos o engañosos en los datos de entrenamiento, comprometiendo el rendimiento del modelo o creando puertas traseras.

Como ejemplo, supongamos que un adversario es capaz de inyectar datos maliciosos en el conjunto de datos de entrenamiento para un modelo utilizado en un software antivirus para determinar si un binario dado es malware. El adversario puede manipular los datos de entrenamiento para establecer efectivamente una puerta trasera (backdoor), permitiéndole crear malware personalizado que el modelo clasifica como benigno.  |
| ML03 | `Ataque de inversión de modelo(Model Inversion Attack)`: Los atacantes entrenan un modelo separado para reconstruir entradas a partir de las salidas del modelo, revelando potencialmente información sensible.

Por ejemplo, modelos que procesan datos médicos, como clasificadores utilizados en la detección de cáncer. Si un modelo inverso puede reconstruir información sobre la información médica de un paciente basándose en la salida del clasificador, la información sensible corre el riesgo de ser filtrada al adversario. Además, los ataques de inversión de modelo son más difíciles de ejecutar si el modelo objetivo proporciona menos información de salida. Por ejemplo, entrenar con éxito un modelo inverso se vuelve mucho más desafiante si un modelo de clasificación solo emite la clase objetivo en lugar de cada probabilidad de salida. |
| ML04 | `Ataque de inferencia de membresía(Membership Inference Attack)`: Los atacantes analizan el comportamiento del modelo para determinar si los datos se incluyeron en el conjunto de datos de entrenamiento del modelo, revelando potencialmente información sensible.

 |
| ML05 | `Robo de modelo(Model Theft)`: Los atacantes entrenan un modelo separado a partir de interacciones con el modelo original, robando así propiedad intelectual. |
| ML06 | `Ataques a la cadena de suministro de IA(AI Supply Chain Attacks)`: Los atacantes explotan vulnerabilidades en cualquier parte de la cadena de suministro de ML. |
| ML07 | `Ataque de aprendizaje por transferencia(Transfer Learning Attack)`: Los atacantes manipulan el modelo base que posteriormente es afinado por un tercero. Esto puede llevar a modelos sesgados o con puertas traseras. |
| ML08 | `Sesgo de modelo(Model Skewing)`: Los atacantes sesgan el comportamiento del modelo con fines maliciosos, por ejemplo, manipulando el conjunto de datos de entrenamiento.

Por ejemplo, supongamos nuestro escenario previamente discutido de un modelo de ML que clasifica si un binario dado es malware. Un adversario podría ser capaz de sesgar el modelo para clasificar malware como binarios benignos al incluir datos de entrenamiento incorrectamente etiquetados en el conjunto de datos de entrenamiento. En particular, un atacante podría agregar su propio binario de malware con una etiqueta `benign` a los datos de entrenamiento para evadir la detección por parte del modelo entrenado. |
| ML09 | `Ataque a la integridad de la salida(Output Integrity Attack)`: Los atacantes manipulan la salida de un modelo antes de su procesamiento, haciendo que parezca que el modelo produjo una salida diferente.

Como ejemplo, considera de nuevo el clasificador de malware de ML. Supongamos que el sistema actúa en función del resultado del clasificador y elimina todos los binarios del disco si se clasifican como malware. Si un atacante puede manipular la salida del clasificador antes de que el sistema subsiguiente actúe, puede introducir malware explotando un ataque a la integridad de la salida. Después de copiar su malware en el sistema objetivo, el clasificador clasificará el binario como malicioso. El atacante luego manipula la salida del modelo a la etiqueta `benign` en lugar de `malicious`. Posteriormente, el sistema subsiguiente no elimina el malware ya que asume que el binario no fue clasificado como malware. |
| ML10 | `Envenenamiento de modelo(Model Poisoning)`: Los atacantes manipulan los pesos del modelo, comprometiendo su rendimiento o creando puertas traseras. |

# Modelo 1 - Clasificación de correos de Spam

Por medio de las palabras que se utilicen en correo el modelo clasifica si es spam o no.

Ejemplo:

message = "Hello World! How are you doing?”

```bash
Predicted class: Ham
Probabilities:
     Ham: 98.93%
    Spam: 1.07%
```

message = Congratulations! You won a prize. Click here to claim: [https://bit.ly/3YCN7PF](https://bit.ly/3YCN7PF)

```bash
Predicted class: Spam
Probabilities:
     Ham: 0.0%
    Spam: 100.0%
```

**Reto:  intentemos engañar al modelo para que clasifique un mensaje de spam como ham**

## **Reformulación (Rephrasing)**

Para reformular el mensaje y que el modelo no lo clasifique como spam, se debe saber cada palabra que porcentaje es una alarma para el modelo.

| Mensaje de entrada | Probabilidad de spam | Probabilidad de ham |
| --- | --- | --- |
| `Congratulations!` | 64,97 % | 35,03 % |
| `Congratulations! You won a prize.` | 99,73 % | 0,27 % |
| `Click here to claim: https://bit.ly/3YCN7PF` | 99,34 % | 0,66 % |
| `https://bit.ly/3YCN7PF` | 87,29 % | 12,71 % |

Por medio de varios intentos con diferntes palabras y combinaciones, se obtiene un mensaje que puede ser un posible phising para poder enegañar a la persona pero que el modelo no lo detecte como spam.

Mensaje : Your account has been blocked. You can unlock your account in the next 24h: [https://bit.ly/3YCN7PF](https://bit.ly/3YCN7PF)

```bash
Predicted class: Ham
Probabilities:
     Ham: 57.39%
    Spam: 42.61%
```

## **Sobrecarga**

Naive Bayes asume que cada palabra contribuye de forma independiente a la probabilidad final.

Es decir, si el mensaje enviado tiene muchas palabras que no son clasificadas como spam y algunas pocas como spam, el clasificador de spam va a suponer por probabilidad de cada palabra idependiente que la mayoria no son spam por lo que el correo completo tampoco lo es.

Ejemplo:

Congratulations! You won a prize. Click here to claim: [https://bit.ly/3YCN7PF](https://bit.ly/3YCN7PF). But I must explain to you how all this mistaken idea of denouncing pleasure and praising pain was born and I will give you a complete account of the system, and expound the actual teachings of the great explorer of the truth, the master-builder of human happiness.

```bash
Predicted class: Ham
Probabilities:
     Ham: 100.0%
    Spam: 0.0%
```

Piensa en sitios web o correos electrónicos que admiten HTML, donde podemos ocultar palabras al usuario en comentarios HTML, mientras que el clasificador de spam puede no ser consciente del contexto HTML y, por lo tanto, seguir basando el veredicto de spam en las palabras contenidas en los comentarios HTML.

## **Manipulando los datos de entrenamiento**

Si es posible acceder a los datos de entrenamiento de modelo, es posible modificar algunos datos para alterar la probabilidad de ciertos correos en especificos.

Ejemplo:

Inicialmente, tenemos que el mensaje:

message = "Hello World! How are you doing?”

```bash
Model accuracy: 94.4%
Predicted class: Ham
Probabilities:
     Ham: 100.0%
    Spam: 0.0%

```

Modificación de datos:

spam,Hello World
spam,How are you doing?

spam,Hello World! How are you
spam,World! How are you doing?

Evaluación final

```bash
Model accuracy: 94.0%
Predicted class: Spam
Probabilities:
     Ham: 0.4%
    Spam: 99.6%
```

Depende de la cantidad de datos que hayan de entrenamiento, mayor es la cantidad de datos que se deben editar o añadir.

# **Ataques a la Generación de Texto (Top 10 de LLM de OWASP)**

OWASP ha publicado una lista de los 10 principales riesgos de seguridad relacionados con el despliegue y la gestión de LLM, específicamente el [Top 10 para Aplicaciones de LLM](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

| ID | Descripción |
| --- | --- |
| LLM01 | `Inyección de Prompts (Prompt Injection)`: Los atacantes manipulan la entrada del LLM, directa o indirectamente, para provocar un comportamiento malicioso o ilegal.

Ejemplos aparentemente benignos, como engañar a un chatbot de soporte técnico basado en un LLM para que proporcione recetas de cocina, también puede llevar a que los LLM generen información deliberadamente falsa, discurso de odio u otro contenido dañino o ilegal. Además, los ataques de inyección de prompts pueden utilizarse para obtener información sensible si esta ha sido compartida con el LLM. |
| LLM02 | `Divulgación de Información Sensible(Sensitive Information Disclosure)`: Los atacantes engañan al LLM para que revele información sensible en la respuesta. |
| LLM03 | `Cadena de Suministro(Supply Chain)`: Los atacantes explotan vulnerabilidades en cualquier parte de la cadena de suministro del LLM. |
| LLM04 | `Envenenamiento de Datos y Modelos(Data and Model Poisoning)`: Los atacantes inyectan datos maliciosos o engañosos en los datos de entrenamiento del LLM, comprometiendo su rendimiento o creando puertas traseras. |
| LLM05 | `Manejo Inadecuado de la Salida(Improper Output Handling)`: La salida del LLM se maneja de forma insegura, lo que resulta en vulnerabilidades de inyección como `Cross-Site Scripting (XSS)`, `Inyección SQL (SQL Injection)` o `Inyección de Comandos (Command Injection)`. |
| LLM06 | `Agencia Excesiva(Excessive Agency)`: Los atacantes explotan el acceso insuficientemente restringido del LLM. |
| LLM07 | `Fuga del Prompt del Sistema(System Prompt Leakage)`: Los atacantes engañan al LLM para que revele las instrucciones del sistema, lo que podría permitir vectores de ataque más avanzados. |
| LLM08 | `Debilidades en Vectores e Incrustaciones(Vector and Embedding Weaknesses)`: Los atacantes explotan vulnerabilidades relacionadas con el manejo o almacenamiento de vectores e incrustaciones (`embeddings`) en aplicaciones de LLM de `Generación Aumentada por Recuperación (RAG)`. |
| LLM09 | `Desinformación(Misinformation)`: Las respuestas generadas por el LLM contienen desinformación, lo que podría derivar en problemas de seguridad. |
| LLM10 | `Consumo sin Límites(Unbounded Consumption)`: Los atacantes introducen entradas en el LLM que resultan en un alto consumo de recursos, lo que podría causar interrupciones en el servicio del LLM o costos elevados. |

# **Marco de IA Segura (SAIF) de Google**

Proporciona principios prácticos para el desarrollo seguro de todo el pipeline de IA, desde la recopilación de datos hasta el despliegue del modelo

## 1. Las 4 áreas

- **Data** → datos de entrenamiento.
- **Infrastructure** → código, almacenamiento, entrenamiento y deployment.
- **Model** → modelo, inputs y outputs.
- **Application** → aplicación, agentes y plugins.

👉 **Piensa:** `Data → Infra → Model → Application`

---

## 2. Los 5 riesgos MÁS importantes

### Prompt Injection

Manipular el prompt para que el modelo haga algo que no debería.

### Sensitive Data Disclosure

Conseguir que el modelo revele información sensible.

### Rogue Actions

Conseguir que un **Agent** ejecute acciones no autorizadas.

### Insecure Integrated Component

Explotar plugins, APIs o herramientas conectadas al modelo.

### Insecure Model Output

Conseguir un output malicioso y que la aplicación lo procese sin validarlo.

---

## 3. Los 2 controles que debes recordar

**Input Validation**

→ Validar/sanitizar lo que entra al modelo.

**Output Validation**

→ Validar/sanitizar lo que sale del modelo.

---

# Para memorizar

> **Como pentester de IA, busca principalmente:**
> 

**¿Puedo manipular el modelo?** → Prompt Injection

**¿Puedo sacar información?** → Data Disclosure

**¿Puedo hacer que actúe?** → Rogue Actions

**¿Puedo atacar sus herramientas?** → Plugins/APIs

**¿La aplicación confía demasiado en el output?** → Insecure Output

# **Red teaming de la IA generativa**

Debido a los aspectos de rápido cambio de los despliegues de IA generativa, los administradores se enfrentan a desafíos únicos. Estos desafíos pueden conducir fácilmente a configuraciones incorrectas o problemas con los despliegues de modelos, lo que puede resultar en vulnerabilidades de seguridad.

Es crucial mantenerse actualizado con los desarrollos actuales en los sistemas de IA generativa para identificar posibles vulnerabilidades de seguridad.

**Naturaleza de caja negra :** Entender por qué un modelo reacciona de una manera determinada a una entrada puede ser muy difícil. Yendo aún más lejos, es todavía más difícil tratar de predecir cómo reaccionará un modelo a una nueva entrada.

## **Robo de modelos (Model Theft)**

Al ejecutar estos ataques, los adversarios pretenden obtener una copia o una estimación de los parámetros del modelo para replicarlo en sus sistemas.

## **Tácticas, Técnicas y Procedimientos (TTP)**

Por ejemplo, un enfoque general para atacar un modelo de IA generativa implica ejecutar el modelo con numerosas entradas y analizar las salidas y respuestas resultantes. Analizar una gran cantidad de pares de entrada-salida ayuda a los adversarios a comprender el funcionamiento interno del modelo y puede ayudar a identificar cualquier posible vulnerabilidad de seguridad. Una buena comprensión de cómo reacciona el modelo a entradas específicas es crucial para llevar a cabo más ataques contra el componente del modelo.

A partir de ahí, los adversarios pueden intentar crear datos de entrada que obliguen al modelo a desviarse de su comportamiento previsto, como las cargas útiles de inyección de prompts (prompt injection)