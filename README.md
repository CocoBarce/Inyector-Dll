# Inyector-Dll
este inyector permite forzar a procesos a cargar un dll (Dynamic Link Library) que originalmente no deberia estar ahi una vez dentro el dll ejecuta codigo en el contexto de el proceso y su memoria ram dedicada
## conceptos importantes
### Procesos
cada procesos tiene su allocated mememory que es mapeada con el sistema operativo es por eso que con WIN32 api (api de windows) podemos leer, reservar, y escrbir espacio de memoria virtual
a su vez cada proceso tiene threads o hilos que permiten ejecutar varias cosas dentro de un mismo proceso es importante saber que ols procesos son identificados por su PID (process id)

### DLL 
un dll es una biblioteca de codigo que carga el o los programas de manera dinamica esto quiere decir que no necesariamente al inicio

### API de windows
la api de windows nos permite interactuar con el funcionamiento del sistema en este proyecto es clave para varias cosas como 
-OpenProcess: Obtiene un handler hacia un proceso para poder interactuar con él
-VirtualAllocEx: Reserva memoria dentro del espacio de otro proceso
-WriteProcessMemory: Escribe datos en la memoria de otro proceso
-CreateRemoteThread: Crea un hilo de ejecución dentro de otro proceso
-LoadLibrary: Carga una DLL en el proceso actual.

### Handles
el handle lo podemos pensar como una llave de acceso que tiene el proceso para usar recursos de nuestro sistema
## Como funciona la tecnica de inyecion 
en primer lugar para que este todo en un archivo se embebe el dll que queremos ejecutar osea el payload en una seccuencia de bytes para que nuestro archivo no necesite dependencias externas
### 1er paso
extraer la dll que esta embebida al inyector con el fin de ser un archivo fisico utilizable este inyector obtiene la ruta del directorio temporal de windows osea %temp% crea un archivo dll con un nombre generico, escribe los bytes embebidos en ese archivo y por ultimo guarda la ruta para usarla luego

### 2do paso
el inyector debe abrir el proceso objetivo(se suelen usar procesos que esten siempre activos) y obtener sus handles luego busca su PID con spanshots luego el sistema operativo crea una entrada de datos que representa este acceso y le devuelve al inyector un numero entero (el handle) que sirve como identificador para despues.

### 3er paso el inyector en si
#### 3.1
una vez tiene la identificacion/handle el inyector usa una funcion de windows para solicitar un bloque de memoria al sistema dentro del espacio virtual del proceso victima el sistema sabe donde buscar gracias al handle y encuentra un bloque de memoria vacio lo marca como reservado y le da a nuestro inyector la direccion virtual

### 3.2
Con la dirección virtual remota y el manejador del proceso, el inyector llama a WriteProcessMemory. El sistema operativo realiza la copia de datos entre espacios de memoria de diferentes procesos de forma segura.
### 3.3 
En este paso, el inyector primero necesita encontrar en memoria la función LoadLibrary, que pertenece a la biblioteca del sistema kernel32.dll. Esta función es la encargada de cargar una DLL dentro de un proceso. Como kernel32.dll está cargada en prácticamente todos los procesos de Windows, su dirección en memoria es compartida o consistente, lo que permite reutilizarla para inyección.

Una vez obtenida esta dirección, el inyector crea un nuevo hilo dentro del proceso objetivo usando CreateRemoteThread. Este hilo se configura para ejecutar directamente la función LoadLibrary, pasándole como parámetro la dirección de memoria (dentro del proceso remoto) donde previamente se escribió la ruta de la DLL (nuestro payload).

Cuando el sistema operativo inicia este hilo, el proceso objetivo comienza a ejecutar LoadLibrary como si fuera propio. Esto provoca que la DLL (nuestro payload) se cargue en su espacio de memoria y que su código se ejecute dentro de ese proceso.

Resultado: la DLL (nuestro payload) queda inyectada y ejecutándose dentro del proceso objetivo, con acceso a su memoria, recursos y permisos.
