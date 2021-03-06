## 15 de Octubre de 2018
- Decido quitar la variable refAlign porque el archivo de entrada siempre tendrá el mismo nombre
- Agrego las variables de MoreFRags y posBInst
- Hago un commit en github en caso de que se dañe el inst2Bin dado que voy a quitar las cosas que no
  debo usar como Edis, y los archivos de salida que usaba Anibal como prueba.
## 1 de Noviembre de 2018
  - Tratar de poner a funcionar una versión secuencial
  - [x] Hacer una copia de mappos, pasar dicha copia al radixsort y luego liberarla
  - Hacer un punto de control cuando se tenga una versión secuencial
  - [x] En el archivo de alineamiento especial para el encoder debo agregar la suma total de errores que generó el ARG
  - Organizar el repositorio del ARG
  - Error Grave entre bases iguales Base1 0 Base2 0: Le debo prestar atención cuando la operacion sea s o S o i.
  - Revisar nuevamente la lectura de los datos desde el archivo de alineamiento
## 4 de Noviembre de 2018
  - El error que tenía era que estaba inicializando el puntero dentro del ciclo, entonces se reescribía con cada pasada del ciclo
  - [x] No olvidar liberar la memoria
  - [x] Aplicar lo del preámbulo
## 6 de Noviembre de 2018
  ### Objetivo: Encontrar porque el BinInst tiene como salida todos sus valores en 0
  - Del preámbulo están saliendo valores
  - Los elementos hasta totalReads tiene el valor del preámbulo
## 10 de Noviembre
  ### Objetivo: Encontrar porque el BinInst tiene como salida todos sus valores en 0
  - Por cuestión de compatibilidad se trabaja a partir de la posición siguiente a la 0, por notación de alineadores
  #### Definición de los campos:
    8 bits que contienen un PRELUDE (hasta el momento, se va a cambiar por que en 8 bits quepan los preludios de dos reads (4 bits por preludio))
    8 bits los menos significativos del offset
    8 bits que se dividen así:
      2 bits que faltan del offset
      1 bits que indica si este es el último error
      3 bits opcode (codigo de la operación)
      2 bits que indican la diferencia entre las bases

  EUREKA
## 19 de Noviembre
  ### Objetivo: Definir el nuevo preludio
  - Definición: Si mal no recuerdo, la idea es sacar los preludios del BinInst y ponerlos en su propio
  arreglo de preludios, en general, por cada 8 bits caben 2 preludios
  - Defino un arreglo de preludios:
    - Recordar que un preludio es un conjunto de 4 bits, el primer bit indica si el siguiente read mapea en la misma posición, los siguientes tres bits corresponden a el código del matching.
    - El arreglo de preludios debe ser de tamaño TotalRead/2, en caso de que el total de reads sea impar,
    entonces se define dicho tamaño como floor(TotalReads/2)+1

### 23 de Noviembre
  ### Objetivo: Comenzar con el paralelismo
  - Planificación ( Distribución de la carga, balanceo de carga dinámica y balanceo de carga estática )
  - En OpenMp está el parámetro schedule
  _PROXIMAS ACTIVIDADES_
  1. HACER PRUEBAS DE QUE LAS COSAS CORRAN CORRECTAMENTE (SEMÁNTICA)

  2. Tiempo esperado, de 2 a tres veces mejor 
  3. Mirar el tema de la distribucion de la carga de trabajo en los reads usando los dos modelos
  4. Verificar los preludios
  
  (Modelo estático: Todos los hilos tienen el mismo número de reads excepto el ultimo)
  (Modelo dinámico: los hilos pelean por chunks, tema de las condiciones de carrera )

### 29 de Noviembre
  ### Objetivo: Extraer y probar la versión secuencial
  - Antes de comenzar la paralelización, organizo la versión secuencial


### RECESO
  ### MOTIVO: Daño del computador pospuso la actividad

# 2019

### 10 de Enero
  ### Objetivo: Retomar el trabajo de grado
  #### Estrategia:
  - [x] Organizar el repositorio de github y acomodarlo a la estrategia de desarrollo de git-flow
  - [x] Finalizar de una vez por todas la versión secuencial, garantizando que funciona para las diferentes cantidades de datos posibles.
  - [x] Comenzar a organizar la documentación.
  ### Puntos clave:
    - El nombre del archivo que se va a procesar esta quemado en el fopen, se debe pasar este parámetro por la línea de comandos.
    - Parece que el ARG no generó los 200.000 reads.
    - Descargue el ARG_ENCODER y volví a generar un archivo de alineamiento, verifiqué que tuviera los 200000 reads, y lo paśe al encoder, en esta ocasión funcionó correctamente. De aquí surge un problema, es probable que bajo ciertas circunstancias la lectura del archivo de alineamiento fracase, hay que verificar dichas circunstancias.

### 17 de Enero
  ### Puntos clave:
    - Se hacen pruebas de los preambulos en la versión secuencial

### 22 de Enero
  ### Puntos Clave:
  ![ESTRUCTURAS](/IMG/ESTRUCTURAS.jpeg)
  - Se completa la versión secuencial, pendiente de pruebas más rigurosas.

### 23 de Enero
  ### Objetivo: Cerrar la versión secuencial, generar la versión paralela estática y hacer pruebas iniciales
  ### Estrategia:
  - [x] Generar los archivos de salida de prueba de la versión secuencial
  - [x] Pasar los archivos a una carpeta de pruebas donde después se van a comparar con los que se generen en las siguientes versiones.
  - [x] Pasar al master la versión secuencial
  - [x] Generar una versión paralela estática
  ### Puntos Clave:
  **PROPUESTA DE GITFLOW**
  **MASTER** Contendrá la versión secuencial
  **DEVELOP** Contendrá la versión paralela estática
  **FEATURE** Contendrá la versión paralela dinámica
  - 
### 24 de Enero
  ### Objetivo: Aplicar las técnicas de paralelización
  ### Estrategia:
  - [x] Descomposición en tareas
  - [x] Descripción de tareas
  - [x] Asignación
  - [x] Correspondencia
  ### Puntos Clave:
  #### DESCOMPOSICIÓN:
  ![DESCOMPOSICIÓN EN TAREAS](/IMG/Descomposicion.jpeg)
  #### DESCRIPCIÓN:
  - **TAREA 0:** Es la tarea más crítica pues requiere sincronizar todos los hilos para que operen correctamente sobre la variable **AuxInd**. Todas las tareas importantes dependen del cálculo correcto del índice
  - **TAREA 1:** Esta tarea es de poca importancia, y no tiene dependencia con el índice auxiliar, en general se trata saber si el siguiente Read mapea en la misma posición que el actual.
  - **TAREA 2:** Determinar el preámbulo, dado que en un solo byte van a estar agrupados dos preámbulo de diferente read, esta tareá se hace crítica pues en las fronteras entre hilos, es posible que el índice se superponga.
  - **TAREA 3:** Determinar el BYTE 1 del Ins2Bin depende del índice auxiliar, pero en gran medida es una tarea independiente y fácil de completar pues ocupa el Byte completo
  - **TAREA 4:** Calcular el BYTE 2 no es tan sencillo, pues requiere que el BYTE 1 ya esté calculado y se tengan los bits sobrantes para calcular agregarlos al BYTE 2, por ello se tiene el mismo problema en las fronteras entre hilos.
  - **TAREA 5:** Actualizar el índice auxiliar tiene su complicación, pues puede ocurrir que los hilos sobrescriban el índice. Se debe tener copias separadas y que en cierto momento se sincronicen.

### 28 de Enero
  ### Objetivo: Terminar de aplicar las técnicas de paralelismo
  ### Puntos clave:
  - Debo separar totalmente las tareas descritas previamente
  - Paso el AuxInd y MoreFrags para ser declaradas dentro de la región de paralelización con el objetivo de que sea una variable privada
  - **TAREA 6:** Añado esta tarea porque también puede ser fuente de error, se trata de liberar la memoria, con esta tarea se debe tener cuidado porque un hilo puede borrar algo que está usando otro hilo 
  - Corrección del punto anterior, de hecho no es posible que se liberen posiciones de memoria incorrectas porque AuxInd es diferente obligatoriamente
  - El sistema de impresión que tengo no va a funcionar, debo esperar a tener todo el preambulo y el BinInst e imprimirlos, ya se agregó otro en el que solo se imprimen los vectores verticalmente

### 29 de Enero
  ### Objetivo: Evaluar el archivo que tiene el error
  - Agregar mostrar en pantallas los límites que toman los hilos ( imprimir istart y iend )
  - Parametrizar la cntidad de hilos
  - Todos los lotes sean par, excepto el ultimo
  ### PROPUESTAS SOLUCION BinINST
  - Hacer todo local y concatenar memoria
  - hacer la sumatoria de los lendesc del chunck pasado ( Calculo paralelo de prefijo exclusivo )

### 31 de Enero
  ### Objetivo: Hacer el desarrollo planteado en la reunión anterior
  ### Estrategia:
  - [X] Parametrizar la cantidad de hilos.
  - [X] Al empezar la región paralela,cada hilo muestra en pantalla los límites que toma
  - [X] Garantizar que para cada hilo, exceptuando el último, el número de preámbulos que toma sea par.
  - [X] Probar
  ### Puntos clave:
  - Se implementa la parametrización y se deja por defecto un valor de 4 hilos
  - 

### 1 de Febrero
  ### Objetivo: Poner en funcionamiento el BinInst para la versión estática
  ### Estrategias:
  - [] Generar un prefix sum paralelo del vector de errores ( Lendesc )
  - [] Evaluar en que puntos cada uno de los hilos debe leer el arreglo anterior
  - [] Cada hilo comenzará a llenar entonces el BinInst desde la posición anterior.
  ### Puntos Clave:
  - Revisar con números raros el tema de los límites

### 11 de Febrero
  ### Objetivo: Completar el prefixsum paralelo
  -> FALLO

### 13 de Febrero
  ### Objetivo: Paralelizar la generación del BinInst
  ### Enfoque 1
  Asumiendo que se soluciona el tema del prefix sum, cada uno de los hilos tiene la información de desde que punto debe escribir en el arreglo, solo consultando en el prefixLendesc cuantos errores existen antes que él
  y luego comenzando escribir en la posición siguiente
  ### Enfoque 2:
  Asumiendo que el prefix lendesc consume mucho tiempo, lo mejor es que cada uno de los hilos defina un subarrelo con su propio BinInst y que al final una ambos, para ello se puede usar realloc para ir variando dinamicamente los arreglos

### 18 de Febrero
  ### Objetivo: Probar la versión estática y comparar con la versión secuencial
  ### Estrategia:
  - [] Organizar el código de la versión estática.
  - [] Organizar el código de la versión secuencial
  - [] Descargar los codigos en el servidor
  - [] Generar archivos de prueba
  - [] Hacer pruebas de funcionalidad

### 20 de Febrero
  ### Objetivo: Comenzar con el desarrollo de la versión dinámica
  **Descripción:** Al programa se pasa como parámetro el tamaños del chunck, los hilos toman porciones del tamaño definido, y al terminar, pasan a tomar otra porción deacuerdo con la última tomada por otro hilo.
  **Consideraciones:**
  * Condiciones de carrera
  * Balanceo de carga
  * Superposición de hilos
  * Última porción

### 25 de Febrero
  ### Objetivo: Organizar la versión secuencial para que trabaje correctamente
  ### Estrategia:
  - [] Depurar el código en su versión secuencial para detectar el error, esto implica aprender a depurar en Visual Studio.
  - [] Cambiar el RadixSort
  - [x] Generar archivos de prueba de 100M
  - [x] Verificar correctitud de los datos
  - [] Hacer un profiling completo de la versión secuencial

### 26 de Febrero
  ### Puntos clave:
  - Generé un archivo de prueba de 100M Reads
  - Corrí la versión secuencial y funcionó sin problemas cuando detecté que el error se generaba por no abrir los archivos donde se guardan los resultados de la prueba de integridad
  - Corrí la versión secuencial generando los archivos para las pruebas de integridad y funcionó correctamente.
  - En mi máquina genero un archivo de prueba con 5M y corro la versión secuencial y funciona bien, generando los archivos de prueba de integridad
