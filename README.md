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

