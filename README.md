# Pruebas de Carga y Rendimiento

Taller de pruebas de rendimiento para el curso Testing y Validación de Software
Maestría en Ingeniería de Software - Universidad de La Sabana.

## Integrantes
- Jose Haider Barreto Oliveros

## Sistema bajo prueba
API REST de registro de votantes `POST /register` — Spring Boot 2.7.18

## SLO definidos
- p95 latencia ≤ 300 ms
- p99 latencia ≤ 800 ms
- Error rate < 1%
- Throughput ≥ 100 req/s

## Escenarios ejecutados
| Escenario | VUs | Duración |
|-----------|-----|----------|
| Baseline | 20 | 5 min |
| Load | 0→200 | 14 min |
| Stress | 200→600 | 10 min |
| Spike | 50→300→50 | 4 min |
| Regression | 20 | 5 min |

## Ejecución

### 1. Levantar el servidor
```bash
cd registraduria
mvn spring-boot:run
```

### 2. Ejecutar pruebas
```bash
k6 run perf/scripts/register_person_k6.js --env SCENARIO=baseline --env BASE_URL=http://localhost:8080
k6 run perf/scripts/register_person_k6.js --env SCENARIO=load --env BASE_URL=http://localhost:8080
k6 run perf/scripts/register_person_k6.js --env SCENARIO=stress --env BASE_URL=http://localhost:8080
k6 run perf/scripts/register_person_k6.js --env SCENARIO=spike --env BASE_URL=http://localhost:8080
k6 run perf/scripts/register_person_k6.js --env SCENARIO=regression --env BASE_URL=http://localhost:8080
```

## Resultados
| Escenario | p95 | Throughput | Error HTTP | SLO |
|-----------|-----|------------|------------|-----|
| Baseline | 6.43 ms | 5,735 req/s | 0% | ✅ |
| Load | 50.14 ms | 6,330 req/s | 0.0006% | ✅ |
| Stress | 93.26 ms | 6,628 req/s | 0.004% | ✅ |
| Spike | 42.45 ms | 6,490 req/s | 0% | ✅ |
| Regression | 4.32 ms | 8,173 req/s | 0% | ✅ |

## Documentación
Wiki: https://github.com/JHBOGameDev/pruebas-carga-rendimiento/wiki