**TP1 – Actividad 04:**

**¿CÓMO IMPLEMENTAR EL PROCESAMIENTO PERIÓDICO DE UNA MEDIANTE UNA TAREA?**

Existen varias formas de implementar un procesamiento periódico mediante una Tarea. La más rudimentaria es utilizando un ciclo NULL ("NULL Loop"). Esta metodología tiene una clara desventaja y es que la tarea de mayor prioridad permanece en estado en ejecución mientras ejecutaba el bucle nulo, provocando inanición (_starving_) en la tarea de menor prioridad al dejarla sin ningún tiempo de procesamiento. 
Para solucionar este problema Barry implementó dos funciones que cumplen esta misma función; `vTaskDelay()` y `vTaskDelayUntil()`, las cuales serán explicadas a continuación:

**_vTaskDelay():_** 

`vTaskDelay()` coloca a la tarea que realiza la llamada en estado **Bloqueado** (_Blocked_) durante un número fijo de interrupciones de _tick_. La tarea no utiliza ningún tiempo de procesamiento mientras se encuentra en el estado Bloqueado, por lo que la tarea solo consume tiempo de procesamiento cuando realmente hay trabajo por hacer.

Es importante tener en cuenta que la función de la API `vTaskDelay()` está disponible solo cuando `INCLUDE_vTaskDelay` se establece en '1' en el archivo `FreeRTOSConfig.h`.

La invocación a la función `vTaskDelay()` se realiza de la siguiente forma:
![vTaskDelay.png](img\vTaskDelay.png)

Parámetro _xTicksToDelay_

Es el número de interrupciones de _tick_ que la tarea que realiza la llamada permanecerá en el estado **Bloqueado** (_Blocked_) antes de hacer la transición de vuelta al estado **Listo** (_Ready_).
    
Por ejemplo, si una tarea llama a `vTaskDelay( 100 )` cuando el contador de _ticks_ está en 10,000, entonces entrará inmediatamente al estado Bloqueado, y permanecerá en el estado Bloqueado hasta que el contador de _ticks_ alcance los 10,100.
La macro `pdMS_TO_TICKS()` se puede utilizar para convertir un tiempo especificado en milisegundos a un tiempo especificado en _ticks_. Por ejemplo, llamar a `vTaskDelay( pdMS_TO_TICKS( 100 ) )` hace que la tarea que realiza la llamada permanezca en el estado Bloqueado durante 100 milisegundos.
![vTaskDelayExample.png](img\vTaskDelayExample.png)

**_vTaskDelayUntil():_**

`vTaskDelayUntil()` es similar a `vTaskDelay()`. Se sabe que al invocar a `vTaskDelay()`, la cantidad de tiempo que la tarea permanece en el estado Bloqueado se especifica, pero el momento en el que la tarea sale del estado Bloqueado es **relativo** al instante en el que se llamó a `vTaskDelay()`. Los parámetros de `vTaskDelayUntil()`, en cambio, especifican el valor exacto del contador de _ticks_ en el cual la tarea que realiza la llamada debe ser movida del estado **Bloqueado** al estado **Listo** (_Ready_).

`vTaskDelayUntil()` es la función de la API que se debe utilizar cuando se requiere un período de ejecución fijo (cuando se desea que la tarea se ejecute periódicamente con una frecuencia fija), ya que el momento en el que la tarea que realiza la llamada se desbloquea es **absoluto**, en lugar de relativo al instante en el que se llamó a la función (como es el caso de `vTaskDelay()`).

Es importante tener en cuenta que la función de la API `vTaskDelayUntil()` está disponible solo cuando `INCLUDE_vTaskDelayUntil` se establece en '1' en el archivo `FreeRTOSConfig.h`.

La invocación a la función `vTaskDelayUntil()` se realiza de la siguiente forma:
![vTaskDelayUntil.png](img\vTaskDelayUntil.png)
Donde,

**`pxPreviousWakeTime`** recibe su nombre partiendo de la suposición de que `vTaskDelayUntil()` se está utilizando para implementar una tarea que se ejecuta periódicamente y con una frecuencia fija. En este caso, `pxPreviousWakeTime` guarda el momento en el que la tarea abandonó por última vez el estado **Bloqueado** (fue 'despertada'). Este tiempo se utiliza como punto de referencia para calcular el momento en el que la tarea debería salir del estado Bloqueado la próxima vez.
La variable a la que apunta `pxPreviousWakeTime` se actualiza automáticamente dentro de la función `vTaskDelayUntil()`; normalmente no debería ser modificada por el código de la aplicación, pero **debe ser inicializada** con el contador de _ticks_ actual antes de su primer uso.

**`xTimeIncrement`** recibe su nombre asumiendo que `vTaskDelayUntil()` se está utilizando para implementar una tarea que se ejecuta periódicamente y con una frecuencia fija, la cual se establece mediante el valor de `xTimeIncrement`. Este parámetro se especifica en '_ticks_'. La macro `pdMS_TO_TICKS()` se puede utilizar para convertir un tiempo especificado en milisegundos a un tiempo especificado en _ticks_.

Un ejemplo de implementación es el siguiente:
![vTaskDelayUntilExample.png](img\vTaskDelayUntilExample.png)

**¿CUÁNDO SE EJECUTARÁ LA TAREA IDLE Y CÓMO SE PUEDE UTILIZAR?**

Siempre debe haber al menos una tarea que pueda entrar al estado en **Ejecución** (_Running_). Para asegurar que este sea el caso, el planificador crea automáticamente una tarea inactiva (tarea **_Idle_**) cuando se llama a `vTaskStartScheduler()`. La tarea _Idle_ hace poco más que quedarse en un bucle, por lo que siempre puede ejecutarse.

La tarea _Idle_ tiene la prioridad más baja posible (prioridad cero), para asegurar que nunca impida que una tarea de la aplicación de mayor prioridad entre al estado en **Ejecución**. Sin embargo, no hay nada que impida a los diseñadores de la aplicación crear tareas en la prioridad de la tarea _Idle_ (y que, por lo tanto, la compartan), si así lo desean. La constante de configuración en tiempo de compilación `configIDLE_SHOULD_YIELD` en `FreeRTOSConfig.h` se puede utilizar para evitar que la tarea _Idle_ consuma tiempo de procesamiento que estaría asignado más productivamente a tareas de la aplicación que también tienen una prioridad de 0. 

Ejecutarse en la prioridad más baja asegura que la tarea _Idle_ haga la transición fuera del estado en **Ejecución** tan pronto como una tarea de mayor prioridad entre al estado **Listo** (_Ready_). Esto se puede ver en el tiempo _tn_ en la siguiente figura,
![TaskIdle.png](img\TaskIdle.png)
donde la tarea _Idle_ es inmediatamente sacada o intercambiada (_swapped out_) para permitir que la Tarea 2 se ejecute en el instante en que la Tarea 2 sale del estado Bloqueado. Se dice que la Tarea 2 ha desplazado o apropiado (_preempted_) a la tarea _Idle_. La apropiación (_preemption_) ocurre automáticamente, y sin el conocimiento de la tarea que está siendo desplazada.

**Nota:** Si una tarea utiliza la función de la API `vTaskDelete()` para eliminarse a sí misma, entonces es esencial que la tarea _Idle_ no sufra de inanición (_starved_) de tiempo de procesamiento. Esto se debe a que la tarea _Idle_ es responsable de limpiar los recursos del kernel utilizados por las tareas que se eliminaron a sí mismas.

_Idle Task Hook Functions_

Es posible añadir funcionalidad específica de la aplicación directamente en la tarea inactiva (tarea **_Idle_**) mediante el uso de una función _hook_ de la tarea _Idle_ (o _callback_ de inactividad), la cual es una función que la tarea _Idle_ llama automáticamente una vez por cada iteración del bucle de la tarea _Idle_.

Los usos comunes para el _hook_ de la tarea _Idle_ incluyen:

* Ejecutar funcionalidad de procesamiento continuo, en segundo plano o de baja prioridad sin la sobrecarga (_overhead_) de memoria RAM que implicaría crear tareas de la aplicación para ese propósito.
* Medir la cantidad de capacidad de procesamiento libre. (La tarea _Idle_ se ejecutará solo cuando ninguna de las tareas de la aplicación de mayor prioridad tenga trabajo que realizar; por lo tanto, medir la cantidad de tiempo de procesamiento asignado a la tarea _Idle_ proporciona una indicación clara del tiempo de procesamiento sobrante).
* Coloca**r** el procesador en un modo de bajo consumo, proporcionando un método fácil y automático para ahorrar energía siempre que no haya procesamiento de la aplicación por realizar (aunque el ahorro de energía que se puede lograr es menor que el obtenido mediante el modo inactivo sin interrupciones de tiempo, o _tick-less idle mode_).

_Limitaciones en la Implementación de las Funciones _Hook_ de la Tarea _Idle__ 

Las funciones _hook_ de la tarea _Idle_ deben adherirse a las siguientes reglas:
* Una función _hook_ de la tarea _Idle_ nunca debe intentar bloquearse o suspenderse a sí misma.
      **Nota:** Bloquear la tarea _Idle_ de cualquier manera podría causar un escenario en el que no haya ninguna tarea disponible para entrar al estado en **Ejecución** (_Running_).
* Si una tarea de la aplicación utiliza la función de la API `vTaskDelete()` para eliminarse a sí misma, entonces el _hook_ de la tarea _Idle_ siempre debe retornar a quien lo llamó dentro de un período de tiempo razonable. Esto se debe a que la tarea _Idle_ es responsable de limpiar los recursos del kernel asignados a las tareas que se eliminan a sí mismas. Si la tarea _Idle_ permanece permanentemente dentro de la función _hook_ de la tarea _Idle_, entonces esta limpieza no podrá ocurrir.

Las Idle Task Hook deben tener un prototipo que reciba un parámetro `void` y devuelva otro igual como el siguiente:
![TaskIdleHook.png](img\TaskIdleHook.png)

**IMPLEMENTACIÓN DE PROCESAMIENTOS PERIÓDICOS CON TAREAS**

Una vez estudiado el funcionamiento de procesamientos periódicos mediante Tareas, se procedió a implementarlos y verificar su eficiencia.
En una primer instancia se hizo correr una programa sencillo con dos tareas. Una de ella se denomina `Task_BTN_1` y la otra `Task_LED`. En una primer instancia no se implementaron de forma periódica y el uso de CPU se distribuyó totalmente y de forma equitativa entre ambas tareas, como se puede observar en la siguiente imagen:

![vTaskDelayUntilNotImplemented.png](img\vTaskDelayUntilNotImplemented.png)

Acto seguido se cambió su implementación para hacer que tuvieran un procesamiento periódico. Ambas se implementaron de forma periódica con la función `vTaskDelayUntil()`. A la tarea `Task_BTN_1` se la periodizó cada 250 ms y a la tarea `Task_LED` cada 500 ms con el objetivo que se ejecute dos veces la tarea `Task_BTN_1` por cada vez que se ejecutaba la tarea `Task_LED`. Para lograr visualizar esto cada tarea imprime por consola que se está ejecutando. Como se puede observar en la siguiente figura, el resultado fue el esperado.

![vTaskDelayUntilNotImplementedConsole.png](img\vTaskDelayUntilNotImplementedConsole.png)

Además, se escaneó el porcentaje de tiempo que cada tarea tiene asignada la CPU y los resultados fueron los siguientes:

![vTaskDelayUntilImplemented.png](img\vTaskDelayUntilImplemented.png)

Entre las dos tareas no llegan a ocupar la CPU el 2% del tiempo. Lo que significa una mejora sustancial al caso anterior.
Como un último experimento se aumentó la prioridad de una de las tareas. Sabiendo que sin la implementación periódica, sólo la tarea de mayor prioridad recibiría la CPU y la otra se moriría de inanición. Como era de esperarse, cuando la tarea de mayor prioridad se bloqueaba por el llamado a `vTaskDelayUntil()`, la otra tarea recibía la CPU y a los fines prácticos el programa funcionó exactamente igual que con las dos tareas con igual prioridad.




