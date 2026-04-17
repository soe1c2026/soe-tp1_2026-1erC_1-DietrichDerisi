**TP1 – Actividad 01 – Paso 02:**

**¿Cómo FreeRTOS asigna tiempo de procesamiento a cada Tarea en una aplicación?**

Asi como esta configurado el proyecto, el FreeRTOS le asigna un 50% a la tarea Task_BTN y un
50% a la otra tarea Task_LED, estos datos son extraidos en base al Run Time%. 

![img1.png](img\img1.png)

**¿Cómo FreeRTOS elige qué Tarea debe ejecutarse en un momento dado?**

FreeRTOS cuando se inicia no se sabe a ciencia  cierta cual tarea de ejecuta primero, puede ser cualquiera no respeta el orden en que fueron creadas en el codigo.
Ahora despues del momento del inicio de la aplicacion, FreeRTOS le da un tiempo a la primera tarea que ya se ejecuto y que el selecciono segun su prioridad, despues de pasado ese tiempo cambia de contexto para ejecutar la siguiente tarea asi se van repartiendo el tiempo de ejecucion( todo esto suponiendo que ambas tareas tienen la misma prioridad de ejecucion y utiliza el metodo _round robin_).

**¿Cómo la prioridad relativa de cada Tarea afecta el comportamiento del sistema?**

La prioridad indica que tiempo de ejecucion maximo tiene asignada cada tarea, siendo la mas alta la que mas tiempo tendra para su ejecucion.
El sistema se ve afectado pues si a una tarea A ,se le da una priodidad mas alta que a otra tarea B ,la A tal vez no le deje tiempo suficiente para terminar de ejecutar la tarea B o la inversa tambien.

**¿Cuáles son los estados en los que puede encontrarse una Tarea ?**

  Hay cuatro estados en FreeRTOS y son **READY**, **RUNNING** , **BLOCKED** y **SUSPENDED**

**¿Cómo implementar Tareas ?**

Primero ver la memoria disponible para usar FreeRTOS, mediante la macro **configMINIMAL\_STACK\_SIZE** ,y dependiendo del espacio de heap suficiente que tengamos podemos crear tareas y demas objetos del SO.
Luego ya creada la funcion que utilizaremos como tarea en este caso **task_led()**, llamamos a la funcion **xTaskCreate()** en la cual tenemos varios parametros que tenemos que completar 

![img6.png](img\img6.png "img6.png")
 
 Dentro de los aprametros mas importantes que debemos completar es el puntero a la funcion , el parametro **TaskFunction_t pxTaskCode** en este caso **task_led** ,en nombre de la tarea **const char * const pcName** en este caso "**Task LED**", el **configSTACK_DEPTH_TYPE usStackDepth** que en nuestro caso es 2 veces el **configMINIMAL\_STACK\_SIZE**(el valor minimo del Stack), el **void * const pvParameters** que en este caso es **NULL** ya  que esta funcion  no admite parametros.
 Siguiendo con las parametros  tenemos el UBaseType_t uxPriority es la prioridad de la tarea en este caso es **tskIDLE\_PRIORITY + 1ul = 1**, y por ultimo el **TaskHandle_t * const pxCreatedTask** que para esta tarea es **\&h\_task\_led**.
 
![img7.png](img\img7.png "img7.png")

**¿Cómo crear una o más instancias de una Tarea ?**

Como se puede apreciar en la imagen anterior se puede crear varias instancias , modificando el parametro const **char * const pcName**  que es el nombre de la tarea 
en este caso podriamos llamar a una tarea **"Task BTN\_1"** ,y modificando el parametro **TaskHandle_t * const pxCreatedTask** que es el Handle de esta tarea 
en particular y su nombre es **\&h\_task\_btn_1**.


**¿Cómo eliminar una Tarea ?**

Para eliminar una tarea se utiliza la funcion vTaskDelete() , por medio de un puntero del tipo **TaskHandle_t** que tiene el Handle de la tarea ,en la imagen siguiente
se ve  el prototipo de la funcion.

![img8.png](img\img8.png "img8.png")
Introduciendo este  **TaskHandle_t** de la tarea se elimina la tarea completa, pero no asi la memoria que esta haya pedido , esta debe ser eliminada de antemano y luego 
debe ser llamada esta funcion para liberar toda esa memoria utilizada.

**TP1 – Actividad 01 – Paso 03:**

 Si cambio la prioridad a la tarea **Task BTN=1** y a **Task LED=2**, al ejecutarlo directamente no llega a leer el boton se queda colgado: 

![img2.png](img\img2.png "img2.png")

y ademas si se observa el **FreeRTOS Task List** se ve que no puede llegar a calcular el Run Time %(en rojo), y el Stack Usage(en verde) se ve como desproporcionado en el uso por un lado el **Task BTN** solo usa el 3.9% y el **Task LED** usa  el 43.9% solo por cambiar la prioridad.

![img3.png](img\img3.png "img3.png")

Ahora si cambiamos las prioridades de las tareas a **Task BTN=2** y a **Task LED=1**,en este caso detecta la pulsacion y se ven los cambios de estado pero en la parte del togle del LED  no funciona que colgado

![img4.png](img\img4.png "img4.png")
Si vemos en el FreeRTOS Task List  se ve que no puede llegar a calcular el Run Time %(en rojo), y el Stack Usage(en verde) se ve como desproporcionado en el uso por un lado el Task BTN  solo usa el 45.3% y el Task LED usa  el 3.9% .

![img5.png](img\img5.png "img5.png")


**TP1 – Actividad 01 – Paso 04:**

Se creo las tres instancias con los nombres **Task BTN\_1**, **Task BTN\_2** y **Task BTN\_3**, al iniciar la aplicacion se colgo no funcionaba , y esto pasada por que la primer tarea llamada
**Task BTN\_1** no podia crearse por que en esta parte del codigo **configASSERT(pdPASS == ret)** justamente daba que **pdPASS != ret** .
Inspeccionando me di cuenta que **configSTACK_DEPTH_TYPE usStackDepth** estaba seteado a **2 x configMINIMAL\_STACK\_SIZE**  duplicaba el valor, lo que sumado a dos tareas iguales mas consumia todo el Heap del sistema. Se procedio a dejar solo **configMINIMAL\_STACK\_SIZE**.
Tambien inspeccionando el IOC se habilito las funciones necesarias para operar en este proyecto , se ingresa en la solapa _Middleware and Software Packs->FREERTOS->Include Parameters_ y aqui se habilita las funciones **vTaskDelete(),** y **xTaskGetCurrentTaskHandle()** y  se desahabilitan todas las demas que no vamos a usar.
Se dejan habilitadas solo **vTaskPrioritySet()**,**uxTaskPriorityGet()** y **pcTaskGetTaskName()**

 ![img9.png](img\img9.png "img9.png")

Ahora compilando ,ejecutando y depurando se logro crear las tres tareas antes de que una sea eliminada

 ![img10.png](img\img10.png "img10.png")

y en la figura siguiente se ya la tarea en este caso la **Task BTN\_3** eliminada.


![img11.png](img\img11.png "img11.png")





