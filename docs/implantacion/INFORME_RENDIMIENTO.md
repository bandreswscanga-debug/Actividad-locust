# Informe de rendimiento y resultados de pruebas de carga

## 1. Objetivo

Evaluar el rendimiento de la aplicación en las tres fases de la actividad, comparando las solicitudes por segundo (RPS), la latencia del percentil 95 (P95), el porcentaje de errores y los usuarios concurrentes. En la Fase 3 también se realiza una prueba de caos para observar el comportamiento del sistema cuando se detiene una réplica del backend.

## 2. Tabla comparativa de resultados

| Fase | Servidor / Arquitectura | Límite de CPU | Usuarios concurrentes | RPS logrados | Latencia P95 (ms) | % Errores |
|---|---|---|---:|---:|---:|---:|
| Fase 1 | Flask Dev (`app.run`) | 0,20 cores | 50 | 32 | 12 | 0 % |
| Fase 2 | Gunicorn (4 workers) | 0,20 cores | 50 | 24,2 | 190 | 0 % |
| Fase 3 | Nginx + 3 réplicas del backend | 0,20 cores por réplica | 150 | 73,1 | 41 | 0 % |

### 2.1. Análisis comparativo

La Fase 1 alcanzó 32 RPS con 50 usuarios concurrentes y una latencia P95 de 12 ms. La Fase 2, con Gunicorn y cuatro workers, registró 24,2 RPS y un P95 de 190 ms. Estos resultados muestran que añadir workers no garantiza por sí solo un mejor rendimiento, ya que también influyen la aplicación, los recursos disponibles y las condiciones de la prueba.

La Fase 3, con Nginx y tres réplicas del backend, alcanzó 73,1 RPS con 150 usuarios concurrentes, un P95 de 41 ms y 0 % de errores registrados. Durante la prueba de caos se detuvo el contenedor `actividad-locust-api-backend-2`, mientras las réplicas 1 y 3 y Nginx permanecieron activos. Locust registró 25.185 solicitudes, 0 fallos, un P99 de 230 ms y un tiempo máximo de 22.891 ms. Aunque fue la configuración con mayor rendimiento medido, la comparación debe considerar que la Fase 3 dispuso de hasta 0,60 cores agregados para los backends, sin contar Nginx, y que es necesario verificar que las solicitudes continuaron durante el intervalo exacto en que la réplica estuvo detenida.