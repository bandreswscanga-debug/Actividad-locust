# Informe de rendimiento y resultados de pruebas de carga

## 1. Objetivo

Evaluar el rendimiento de la aplicación en las tres fases de la actividad, comparando las solicitudes por segundo, el tiempo de respuesta del percentil 95 (P95) y el porcentaje de errores. En la Fase 3 también se realiza una prueba de caos para observar el comportamiento del sistema cuando una réplica del backend se detiene.

## 2. Resultados de las pruebas

| Fase | Configuración | Usuarios | RPS | P95 (ms) | Errores |
|---|---|---:|---:|---:|---:|
| Fase 1 | Flask, servidor de desarrollo | Pendiente | Pendiente | Pendiente | Pendiente |
| Fase 2 | Gunicorn con 4 workers | Pendiente | Pendiente | Pendiente | Pendiente |
| Fase 3 | Nginx y 3 réplicas del backend | 150 configurados; 149 activos en la captura | 73,1 | 41 | 0 % |

Los valores de las fases 1 y 2 se completarán con los resultados reales obtenidos por los compañeros responsables de esas pruebas.

## 3. Prueba de caos de la Fase 3

Durante la prueba se detuvo el contenedor `actividad-locust-api-backend-2`. Los contenedores de las réplicas 1 y 3 y el contenedor de Nginx permanecieron activos.

En la captura de Locust se registraron 25.185 solicitudes, 0 solicitudes fallidas, un P95 de 41 ms y un rendimiento agregado de 73,1 solicitudes por segundo. Estos resultados corresponden a la captura tomada durante la prueba; por sí solos no demuestran que todas las solicitudes se procesaran mientras la réplica estaba detenida, por lo que deben interpretarse junto con la evidencia de Docker y el intervalo exacto de la prueba.

## 4. Análisis técnico

Gunicorn permite atender solicitudes mediante varios workers, mientras que Nginx actúa como punto de entrada y distribuye las solicitudes entre las réplicas configuradas del backend. Esta arquitectura puede mejorar la capacidad de atención y evitar que una sola instancia sea el único punto de procesamiento. El resultado definitivo de la comparación dependerá de los valores medidos en las tres fases y de las condiciones de cada prueba.

En la prueba de caos, las dos réplicas restantes y Nginx continuaron activos después de detener una instancia. El 0 % de errores registrado es favorable; sin embargo, para confirmar la tolerancia a fallos es necesario verificar que Locust siguió enviando solicitudes durante todo el intervalo con la réplica detenida. También se debe considerar el tiempo máximo registrado de 22.891 ms, que indica que al menos una solicitud tuvo una demora considerable.

## 5. Evidencias

- Captura de Docker con la réplica 2 detenida y las réplicas 1 y 3 activas.

- Captura de Locust con los resultados de la prueba de carga y el porcentaje de fallos.

## 6. Conclusión

La Fase 3 registró 25.185 solicitudes sin fallos en la captura analizada, con un P95 de 41 ms y un rendimiento agregado de 73,1 solicitudes por segundo. Los resultados de las fases 1 y 2 permitirán establecer una comparación completa y determinar las diferencias de rendimiento entre las configuraciones.
