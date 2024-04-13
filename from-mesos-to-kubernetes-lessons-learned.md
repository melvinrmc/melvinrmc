# From Mesos to Kubernetes: Lessons learned

## Entender el problema
El primer paso es entender el problema, por lo que el contexto es muy importante.

Mantener un cluster activo es muy costoso en terminos monetarios. Costos en infraestructura y costos en licenciamiento.

Como en cualquier industria siempre habrá una politica de reducción de costos.

Cual es la necesidad: se gasta demasiado en licencias.
Qué hay que hacer: bajar costos de licenciamiento.
Cómo: Cambiar software privativo a OpenSource.

Parece una obviedad pero es importantísimo para definir el objetivo del proyecto, el alcance y la decisiones futuras.

Objetivo del proyecto: Movernos de Mesosphere a una solución más económica. (Tiene sentido)

## Lecciones Aprendidas
### No subestimar el cambio

El cambio en sí tiene su dificultad. Pero se multiplica por el numero de componentes con los cuales interactúa.

```
+------+                    +----------+  
|  C1  |-------+    +----+  |  C2      |  
|      |       |    |       |          |  
+------+       |    |       +----------+  
               |    |                     
             +-|----|--+                  
         +-- |  CORE   |----------+       
    +----|   |         |          |       
    |        +---------+          |       
+-------+         |         +-----|------+
|  C3   |         |         |     C4     |
|       |         |         |            |
+-------+     +--------+    +------------+
              |  C..N  |                  
              |        |                  
              +--------+                  
```


Lo importante de reemplazar un componente dentro de un sistema no es el sistema en sí, sino en la interacción de todos los componententes contra el sistema que se va a reemplazar.

Reserva un espacio de tiempo para pruebas de integración.

### Identificar las aplicaciones

A pesar de tener aplicaciones contenerizadas, build-once and run-everywhere, sí, está bien, es lo ideal, pero en la práctica siempre habrán matices.

La mayor dificultad que enfrentamos fue asignar el correcto número de [cpu](https://docs.docker.com/config/containers/resource_constraints/#cpu) contra el [cpu limit](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu)
de k8s. Cuando la infraestructura cambia, el cloud provider cambia, incluso cuando una maquina se destruye y una nueva se genera, en algunas ocasiones se disparaba el 
[CPU throttling](https://sysdig.com/blog/troubleshoot-kubernetes-oom/#:~:text=Register%20now-,Kubernetes%20CPU%20throttling,Limit%20set%20on%20the%20container.).

Cómo lo resolvimos: tomamos dos o tres aplicaciones representativas, realizamos pruebas de estress y ajustando el cpu limit. Se realizó una tabla de conversión equivalente entre una infraestructura y otra que aplicamos al momento de la migración de aplicaciones masivas.

### Hacer los ajustes: Los estándares son importantes.

Los estándares son lo más importante.
A nivel operativo, establecer lineamientos y lo más importante seguirlos, facilita cualquier movimiento de aplicaciones entre cluster, nubes o máquinas.
Si no tienen estándares, establecerlos, realizar los ajustes a manera de que cada traslado de aplicación se vuelva un trabajo operativo y trabajo en serie.
Tener estándares te permite crear recetas que luego puede seguir cualquier colaborador.

### Zero Downtime: Un Transplante a corazón abierto
Me gusta pensar esto como un trasplante a corazón abierto. La única constante es el cambio, dijo Séneca, 
y si el cambio es lo más importante las soluciones se diseñan para tolerar el cambio.

Los ensayos son importantes, garantizar 

```
                   +-------------------+                                                                                           
                   |                   |                                                                                           
                   |     ApiGateway    |                                                                                           
                   +---------|---------+                                                                                           
                             |                                                                                                     
                             |------------------------------------------------------------+                                        
                             |                                                            |                                        
                             |                                                            |                                        
                    +--------+--------+                                                   |                                        
            +-----  |  LoadBalancers  ----------+                                +--------|--------+                               
            |       +-----------------+         |                         +-------LoadBalancers    ---------+                      
            |                                   |                         |      +-----------------+        |                      
            |                                   |                         |                                 |                      
            |                                   |                         |                                 |                      
   +---------------+                   +---------------+          +-------|-------+                 +-------|-------+              
   |mesos-site-1   |                   |  mesos-site-2 |          |    k8s-site-1 |                 |   k8s-site-2  |              
   |               |                   |               |          |               |                 |               |              
   +---------------+                   +---------------+          +---------------+                 +---------------+              
```
### Desvio de Trafico

No todo en la vida se puede prever, pero hay que confiar. El riesgo no se evita, sino que se gestiona. La ultima decisión era la más importante, el cambio sería gradual, o el cambio
sería un bigbang.
Más de 250 aplicaciones gradualmente llevaría un año en completarse. Evaluamos el riesgo, lo mitigamos, y decidimos.

### Monitoreo

Como dije anteriormente, los estándares son importantes.

Estandarizar aplicaciones, estandarizar logs, estandarizar la recopilacion de logs, permite construir métricas.
La estandarización nos permitió "crear" o replicar mejor dicho la infraestructura.
