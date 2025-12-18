Este programa implementa un sistema simple de preguntas y respuestas usando como fuente un archivo PDF. La idea es que el usuario pueda subir un PDF, hacer una pregunta por consola, y que el sistema responda utilizando el contenido del documento.

¿QUÉ HACE EL PROGRAMA?

A grandes rasgos, el programa:

- Lee un PDF y extrae su texto completo.
- Limpia el texto para dejarlo en un formato consistente, es decir, convierte a minúsculas, elimina caracteres especiales y normaliza espacios/saltos de línea.
- Divide el texto en chunks (trozos/fragmentos) para poder buscar información dentro del documento de forma eficiente.
- Permite que el usuario escriba una pregunta por consola y selecciona los chunks más parecidos usando TF-IDF + similitud coseno, tomando el chunk más relevante como contexto, para así generar una respuesta con un modelo.

El programa se divide en tres etapas principales:
- Extracción/limpieza/conteo
- Búsqueda semántica simple
- Generación de respuesta

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

A continuación, se explicará cada etapa de manera detallada:

ÍTEM 1: Extracción/Limpieza/Conteo

Lo primero, es instalar e importar de librerías necesarias (por ejemplo pdfplumber, re, nltk, sklearn, transformers). Luego de esto, se hace la lectura del PDF, abriendo el archivo con pdfplumber. Luego, se recorre página por página y se concatena todo el texto en una sola variable.
Lo siguiente es la limpieza del texto, en donde se convierte el texto a minúsculas (con lower()). Luego se eliminan caracteres no alfabéticos usando expresiones regulares (re.sub) y se normalizan espacios y saltos de línea para evitar formatos que no puedan ser consistentes. Posterior a esto se hace el procesamiento de palabras y stopwords, en donde se separa el texto en palabras (usando split), se cargan las stopwords con la herramienta nltk, lo cual permite usar listas que ya están definidas, esto filtrado por idioma (en este caso de usa inglés, pero esto es configurable). Lo siguiente es filtrar las stopwords, para así pasar al conteo de palabras frecuentes. Para ello, se usa Counter para obtener las palabras más frecuentes una vez filtradas (es decir, sin las stopwords). Esta etapa sirve para garantizar que el texto quedó limpio y se pueda procesar bien, además de entregar información útil de verificación, como la cantidad de palabras, las palabras más frecuentes, etc.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ÍTEM 2: Búsqueda semántica simple

Se comienza con la creación de chunks, en donde se define un tamaño fijo (en este caso 500 caracteres). Luego, se recorre el texto limpio y se divide en fragmentos del mismo tamaño, para que así se impriman resultados para verificar el número de chunks creados, el largo del primero, tener una muestra, etc.
El paso siguiente es hacer el cálculo de similitud (TF-IDF + coseno). Para ello, se define una función que recibe los chunks, la query (pregunta) y un k (que son la cantidad de resultados). Para esto, se crea una lista chunks + [query] para construir un vocabulario común. Posterior a esto, se vectoriza el texto con TF-IDF, donde su función es convertir texto en vectores numéricos, ya que los modelos entienden y trabajan con números. Luego de esto, se calcula la similitud coseno, el cual sirve para medir qué tan parecidos son los vectores entre sí (la pregunta del usuario y los chunks).
Lo siguiente es utilizar argsort, para ordenar los chunks por similitud, y luego recuperar el top-k.
Después se pasa a la pregunta por consola, en donde el usuario debe escribir la pregunta que desee. El programa va a imprimir el top 3 de chunks más similares a la pregunta, mostrando el índice del chunk y el puntaje de similitud.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ÍTEM 3: Generación de respuesta

Lo primero que se hace es cargar el modelo, donde el utilizado fue el google/flan-t5-small. Una vez hecho esto, se cargan AutoTokenizer y AutoModelForSeq2SeqLM, para así pasar al input de la función de generación, que funciona de la siguiente forma:
1. Se construye un prompt, el cual se hace la pregunta y se da contexto.
2. Se tokeniza el prompt y se define un máximo de tokens (con truncado si el texto es largo).

Para el output, se controla el largo de la respuesta actualizando en una variable los nuevos tokens, y se usa do_sample=False, donde su función es hacer una respuesta estable y que no varíe o altere entre ejecuciones.
Luego de esto, se transforma los tokens a texto y se eliminan tokens especiales que puedan haber en el modelo.
Para terminar, se selecciona el chunk más relevante (por ejemplo el top 1), se genera la respuesta y se imprime junto a la pregunta. La idea de esta etapa es que el modelo pueda intentar responder a la pregunta del usuario solo usando contexto del PDF que se subió al principio, es decir, sin inventar.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

PASO A PASO PARA UTILIZAR EL PROGRAMA:

A continuación, se dará el paso a paso para que el usuario pueda usar el programa sin problemas.

1. Coloca tu PDF en la parte donde de suben los archivos (ícono de carpeta en la parte izquierda del colab), esto con el nombre configurado (por ejemplo paper.pdf).
2. Ejecuta los bloques en orden (los bloques están enumerados dentro del programa, entonces se debe ejecutar desde el bloque 1 hasta el 17).
3. Cuando el programa lo solicite (bloque 14), escribe tu pregunta por consola, ejemplo:
   - Escribe tu pregunta: "What is the transformer architecture?"
   El programa te devolverá la pregunta con el top 3 de los chunks más relevantes para responder a tu pregunta.
4. Sigue ejecutando cada bloque del programa hasta llegar al bloque final (bloque 17), el cual te mostrará la pregunta que hiciste con la respuesta que generó el modelo.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

COSAS A CONSIDERAR

- Idioma de stopwords: Se puede cambiar el idioma de stopwords según el PDF con el que trabaje el usuario (ejemplo: si el PDF está en español, en el bloque 9, ponen "spanish" en esta línea: IDIOMA_STOPWORDS = "spanish"). Se pueden usar varios idiomas, como "english", "spanish", "portuguese", etc.
En caso que el PDF está en un idioma distinto al configurado en stopwords, el filtrado puede ser incorrecto.

- Si el chunk seleccionado no contiene la respuesta, el modelo puede generar una respuesta poco útil o incompleta.
