# Informe de rendimiento y resultados de pruebas de carga

## 1. Objetivo

Evaluar el rendimiento de la aplicación en las tres fases de la actividad, comparando las solicitudes por segundo (RPS), el tiempo de respuesta del percentil 95 (P95), el porcentaje de errores y la cantidad de usuarios concurrentes. En la Fase 3 también se realiza una prueba de caos para observar el comportamiento del sistema cuando una réplica del backend se detiene.

## 2. Tabla comparativa de resultados

| Fase | Servidor / Arquitectura | Límite de CPU | Usuarios concurrentes | RPS logrados | Latencia P95 (ms) | % Errores |
|---|---|---|---:|---:|---:|---:|
| Fase 1 | Flask Dev (`app.run`) | 0,20 cores | 50 | 32 | 12 | 0 % |
| Fase 2 | Gunicorn (4 workers) | 0,20 cores | 50 | 24,2 | 190 | 0 % |
| Fase 3 | Nginx + 3 réplicas del backend | 0,20 cores por réplica | 150 | 73,1 | 41 | 0 % |

### 2.1. Análisis comparativo

La Fase 1, basada en el servidor de desarrollo de Flask, alcanzó 32 solicitudes por segundo con 50 usuarios concurrentes y una latencia P95 de 12 ms. En la Fase 2, Gunicorn con cuatro workers registró 24,2 solicitudes por segundo y una latencia P95 de 190 ms. Aunque Gunicorn permite gestionar solicitudes mediante varios procesos de trabajo, los resultados muestran que añadir workers no garantiza por sí solo un mejor rendimiento. El comportamiento también depende de la aplicación, de los recursos disponibles y de las condiciones de la prueba.

La Fase 3, con Nginx y tres réplicas del backend, alcanzó 73,1 solicitudes por segundo con 150 usuarios concurrentes, un P95 de 41 ms y 0 % de errores registrados. Fue la configuración con mayor rendimiento medido y permitió probar una cantidad superior de usuarios concurrentes. Sin embargo, hay una diferencia importante en los recursos: las fases 1 y 2 tenían un límite de 0,20 cores, mientras que la Fase 3 asignó 0,20 cores a cada una de las tres réplicas, lo que representa hasta 0,60 cores agregados para los backends, sin contar Nginx. Por esta razón, la mejora observada debe interpretarse teniendo en cuenta tanto la arquitectura como los recursos adicionales asignados.

## 3. Prueba de caos de la Fase 3

Durante la prueba se detuvo el contenedor `actividad-locust-api-backend-2`. Las réplicas 1 y 3 y el contenedor de Nginx permanecieron activos.

En la captura de Locust se registraron los siguientes resultados:

- Usuarios configurados: 150.
- Usuarios activos mostrados en la captura: 149.
- Solicitudes totales: 25.185.
- Solicitudes fallidas: 0.
- Porcentaje de errores: 0 %.
- Rendimiento agregado: 73,1 solicitudes por segundo.
- Latencia P95: 41 ms.
- Latencia P99: 230 ms.
- Tiempo máximo registrado: 22.891 ms.

El porcentaje de errores fue favorable durante la prueba registrada. No obstante, para confirmar que la arquitectura toleró correctamente la caída de una réplica, es importante verificar que las solicitudes continuaron ejecutándose durante todo el intervalo en que el contenedor estuvo detenido. Los resultados también muestran que, aunque el P95 fue de 41 ms, al menos una solicitud presentó una demora considerable, con un máximo de 22.891 ms.

## 4. Análisis técnico de la arquitectura

Flask Dev permite realizar pruebas básicas de la aplicación, mientras que Gunicorn proporciona varios workers para atender solicitudes mediante procesos de trabajo. Nginx, por su parte, actúa como punto de entrada y puede distribuir las solicitudes entre las réplicas del backend configuradas en el servicio. Esta arquitectura permite probar una estrategia de escalabilidad horizontal y reducir la dependencia de una sola instancia de aplicación.

En las mediciones obtenidas, la Fase 3 presentó el mayor rendimiento, con 73,1 RPS y 0 % de errores, frente a los 32 RPS de la Fase 1 y los 24,2 RPS de la Fase 2. La comparación no es completamente equivalente porque la Fase 3 utilizó 150 usuarios y dispuso de mayores recursos agregados para los backends. Por ello, los resultados respaldan el comportamiento observado en esta configuración, pero no permiten atribuir toda la mejora exclusivamente a Nginx o al número de réplicas.

## 5. Evidencias

### 5.1. Docker: réplica detenida

La captura muestra el estado de los contenedores durante la prueba de caos. La réplica 2 aparece detenida, mientras que las réplicas 1 y 3 y Nginx permanecen activos.

![Docker: réplica detenida](evidencias/docker-replica-detenida.png)

### 5.2. Locust: resultados de la prueba de carga

La captura muestra las métricas registradas por Locust durante la prueba, incluyendo el total de solicitudes, el rendimiento y el porcentaje de fallos.

![Locust: resultados de la prueba de carga](evidencias/locust-prueba-caos.png)

## 6. Conclusión

Los resultados muestran diferencias entre las tres configuraciones evaluadas. La Fase 1 obtuvo 32 RPS y un P95 de 12 ms; la Fase 2 alcanzó 24,2 RPS y un P95 de 190 ms; y la Fase 3 registró 73,1 RPS y un P95 de 41 ms con 150 usuarios concurrentes. Las tres pruebas reportaron 0 % de errores según los datos suministrados.

La arquitectura de la Fase 3 presentó el mayor rendimiento medido y permitió realizar una prueba de caos con una de las réplicas detenida. Las evidencias documentan el estado de los contenedores y las métricas de Locust. Para interpretar los resultados correctamente, se deben considerar las diferencias en usuarios concurrentes y límites agregados de CPU, así como verificar el comportamiento de las solicitudes durante el intervalo exacto de la prueba de caos.
