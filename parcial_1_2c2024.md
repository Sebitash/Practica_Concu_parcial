# 16/10/2024
https://www.notion.so/lihuencarranza/Parcial-1-24C2-11dec673880f80e1809eec0f46a9c68d
## ejercicio 1: identificar busy wait
![alt text](/fotos_parcial1_2c2024/image-1.png)
#### BUSY WAIT
    El busy wait es una técnica en programación concurrente donde un proceso o hilo espera activamente a
    que ocurra una condición específica que no depende del alcance del proceso (como la liberación de un
    recurso o la llegada de un evento) ejecutando repetidamente un bucle de verificación, sin liberar el
    procesador.
    En lugar de pasar el control al sistema operativo para poner el proceso en espera, el programa sigue
    ejecutando instrucciones inútiles mientras comprueba continuamente la condición de espera. Esto
    consume recursos del procesador de manera ineficiente porque el proceso está ocupando tiempo de
    CPU sin realizar un trabajo productivo.
    No tengo que hacer un while se cumpla cierta condicion, tengo que dejar que de eso se encargue el SO.
    Ejemplo típico de busy wait:

    Esta técnica puede ser necesaria en algunas situaciones, pero se considera ineficiente. En su lugar, es
    preferible usar mecanismos de sincronización como semáforos, mutex o barreras, que permiten suspender
    un hilo de manera eficiente hasta que se cumpla la condición.
### A
    rand::thread_rng().gen() genera un numero random
    SI, es un busy wait. El thread se despierta multiples veces para preguntar si se consiguio la conexión en vez de esperar. 
    Puede llegarse el caso que el thread se despierte y no haya conexión, por lo que vuelve a dormir y despierta nuevamente, generando un bucle sin sentido.

### B
    
    Es un busy wait porque está comprobando constantemente si hay algún item expirado. 
    No utiliza ningún método de sincronización que dependa del tiempo de expiración.
    Por lo tanto está haciendo numerosas ejecuciones innecesarias.

### C
    No es busy wait, ya que tenemos un RWLock que espera de forma eficiente a que el recurso este disponible para utilizarlo y liberarlo a continuacion.

## ejercicio 2: red de petri
![alt text](/fotos_parcial1_2c2024/image-2.png)
![alt text](/fotos_parcial1_2c2024/image_petri_sin_preferencia.png)
![alt text](/fotos_parcial1_2c2024/image_petri_con_preferencia.png)
## ejercicio 3
![alt text](/fotos_parcial1_2c2024/image-3.png)
![alt text](/fotos_parcial1_2c2024/image_actores.png)
## ejercicio 4
![alt text](/fotos_parcial1_2c2024/image-4.png)
### A
ES FALSO, los hilos y las tareas asincronica comparten el mismo espacio de memoria del mismo proceso. Pero, para poder acceder a la memoria de forma segura, cada hilo o tarea del proceso, tiene que utilizar metodos de sincronización.

### B
ES FALSO, las tareas asincronas solo pueden ser ejecutados por el proceso que lo invoco en su ejecucion. El sistema operativo solo puede parar hilos, no procesos.
### C
ES FALSO, las tareas asincronas en rust no tienen un stack propio, son stackless. Son tareas que se ejecutan en el stack del hilo que las invoca. Y mantienen su estado utilizando una máquina de estado.
### D
ES FALSO, El proceso que utiliza hilos de procesamiento puede tardar mas o igual al que utiliza un conjunto de tareas asincronas.
## ejercicio 5
![alt text](/fotos_parcial1_2c2024/image-5.png)
### A: convertir un conjunto extenso de archivos .doc a .pdf
ES VECTORIZACION, ya que se puede dividir el trabajo en partes independientes. Cada hilo puede encargarse de convertir un archivo diferente sin necesidad de esperar a que otro hilo termine.
### B: backend para una aplicacion de preguntas y respuestas.
Programación asincronica. tenes que esperar a que el usuario haga una pregunta y luego responderla. No se puede dividir el trabajo en partes independientes.
### C
estado mutable compartido/RwLock, todas las requests van a acceder ahi, pero tienen que entrar de una a la vez para no pisarse entre ellas. 
### D
ES VECTORIZACION, para ejecutar el modelo y minimizar el tiempo de espera. Cada hilo puede encargarse de una parte del modelo y luego combinar los resultados. 