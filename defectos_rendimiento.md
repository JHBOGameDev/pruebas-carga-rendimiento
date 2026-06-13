# Registro de Defectos — Pruebas de Carga y Rendimiento

Curso: Testing y Validación de Software  
Proyecto: Pruebas de Carga y Rendimiento  
Equipo: Jose Haider Barreto Oliveros
Fecha: Junio 2026

---

## Introducción

Este documento recopila los defectos identificados durante la ejecución
de pruebas de rendimiento (Baseline, Load, Stress, Spike y Regresión).
Cada defecto se documenta para garantizar trazabilidad, análisis técnico
y propuesta de mejora.

---

# Formato 1: Lista detallada

## Defecto PERF-01 — Dataset insuficiente genera alta tasa de DUPLICATED

- Capa afectada: Datos de prueba / Aplicación
- Escenario: Baseline, Load, Stress, Spike, Regression
- SLO definido: register_failed < 1%
- Resultado esperado: Registros VALID en más del 99% de las iteraciones
- Resultado obtenido: DUPLICATED rate = 44% (baseline), 31% (load), 19% (stress)

### Evidencia
```
register_failed: rate=0.44 (baseline)
register_failed: rate=0.31 (load)
register_failed: rate=0.19 (stress)
```

### Impacto
El check `body VALID` falla en más del 19% de las iteraciones en todos
los escenarios, superando el SLO de register_failed < 1%.

### Causa probable
- El dataset `persons.csv` tiene solo 5 personas.
- Con 20-600 VUs ejecutando millones de iteraciones, los IDs se agotan
  rápidamente y el sistema devuelve DUPLICATED en lugar de VALID.

### Estado
Abierto

### Prioridad
Alta

---

## Defecto PERF-02 — Degradación de latencia del 680% bajo carga de 200 VUs

- Capa afectada: Aplicación / Servidor
- Escenario: Load Test (0→200 VUs)
- SLO definido: p95 < 300ms
- Resultado esperado: Latencia similar al baseline
- Resultado obtenido: p95 = 50.14ms (+680% vs baseline de 6.43ms)

### Evidencia
```
http_req_duration baseline: p(95)=6.43ms
http_req_duration load:     p(95)=50.14ms
Incremento: +680%
```

### Impacto
Aunque el SLO de 300ms se cumple, la degradación progresiva indica
riesgo de incumplimiento en entornos con mayor carga o con BD real.

### Causa probable
- Servidor Spring Boot usa H2 en memoria con conexión única.
- Bajo alta concurrencia las peticiones se encolan esperando acceso
  al recurso compartido.

### Estado
Abierto

### Prioridad
Media

---

## Defecto PERF-03 — Picos de latencia máxima superiores a 1,900ms

- Capa afectada: Servidor de aplicación / JVM
- Escenario: Load (max=1,949ms), Stress (max=1,784ms)
- SLO definido: p99 < 800ms
- Resultado esperado: Latencia máxima controlada
- Resultado obtenido: max = 1,949ms en Load, 1,784ms en Stress

### Evidencia
```
http_req_duration load:   max=1949.7ms
http_req_duration stress: max=1784.9ms
```

### Impacto
Usuarios en el percentil más alto experimentan tiempos de respuesta
de casi 2 segundos, afectando la experiencia bajo demanda alta.

### Causa probable
- Contención de hilos en Tomcat bajo alta concurrencia.
- Configuración por defecto de max-threads=200 insuficiente.

### Estado
Abierto

### Prioridad
Alta

---

# Formato 2: Tabla de seguimiento

| ID | Escenario | Resultado Esperado | Resultado Obtenido | Estado | Prioridad |
|---|---|---|---|---|---|
| PERF-01 | Todos | register_failed < 1% | 19-44% DUPLICATED | Abierto | Alta |
| PERF-02 | Load | p95 similar a baseline | p95 +680% (50ms) | Abierto | Media |
| PERF-03 | Load/Stress | max < 800ms | max = 1,949ms | Abierto | Alta |

---

## Convenciones de Estado

- **Abierto:** Defecto identificado sin corrección aplicada.
- **En progreso:** En proceso de corrección.
- **Resuelto:** Corregido y validado con nuevas pruebas.

---

Universidad de La Sabana — Facultad de Ingeniería  
Curso: Testing y Validación de Software (2025-1)