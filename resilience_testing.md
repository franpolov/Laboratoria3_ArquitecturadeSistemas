# Resilience Testing Plan

Este documento describe las pruebas de resiliencia realizadas sobre la arquitectura DeliverEasy utilizando **Gremlin Agent** y mecanismos nativos de Kubernetes.  
El objetivo es validar el comportamiento del sistema frente a fallos controlados y confirmar que cumple con los objetivos de continuidad (RTO/RPO).

---

## 1. Objetivos de las Pruebas

- Evaluar la resistencia del servicio Orders API ante fallos en la base de datos.
- Validar la capacidad del clúster EKS para recuperarse automáticamente.
- Detectar cuellos de botella o dependencias críticas.
- Verificar que las métricas y logs necesarios están disponibles para diagnóstico.
- Confirmar que el sistema se comporta de manera consistente bajo estrés.

---

## 2. Entorno de Pruebas

- **Ubicación del Agente:**  
  Gremlin Agent instalado en los nodos del cluster EKS.

- **Servicios evaluados:**  
  - Orders API (Pod en EKS)  
  - DBService  
  - EC2 DB Clone  
  - Load Balancer (tráfico entrante)  

- **Monitoreo habilitado:**  
  - CloudWatch Metrics  
  - CloudWatch Logs  
  - CloudTrail  
  - VPC Flow Logs  
  - Health checks del ALB  

---

## 3. Experimentos Ejecutados

### 3.1 Ataque de Latencia (Latency Attack)
**Propósito:** Simular lentitud en la base de datos para validar el comportamiento de la API bajo degradación.

**Configuración:**
- Latencia agregada: 150–300 ms
- Target: Pod con DBService
- Duración: 3 minutos

**Resultados esperados:**
- Aumento controlado de tiempos de respuesta.
- Logs correctamente enviados a CloudWatch.
- Health checks del ALB permanecen OK.

---

### 3.2 Ataque de Interrupción del Proceso (Process Killer)
**Propósito:** Simular la caída repentina del proceso PostgreSQL en EC2 Clone.

**Configuración:**
- Target: Instancia EC2-Clone
- Acciones: kill -9 al proceso principal de PostgreSQL

**Resultados esperados:**
- Orders API retorna errores temporales (5xx).
- CloudWatch registra reconexiones automáticas.
- EKS sigue funcionando sin degradación interna.
- La DB se recupera automáticamente mediante systemd.

---

### 3.3 Ataque de Pérdida de Paquetes (Packet Loss)
**Propósito:** Validar el comportamiento de red entre EKS y la base de datos.

**Configuración:**
- Packet Loss: 20%
- Interface: conexión entre Pod y SG del EC2 Clone
- Duración: 2 minutos

**Resultados esperados:**
- DBService muestra reintentos de conexión.
- No se produce downtime total.
- ALB sigue enroutando correctamente.

---

### 3.4 Ataque de Consumo de CPU (CPU Stress)
**Propósito:** Simular sobrecarga de un Pod para activar autoescalado.

**Configuración:**
- CPU Load: 80%
- Target: Pod del Orders API
- Duración: 5 minutos

**Resultados esperados:**
- HPA (Horizontal Pod Autoscaler) detecta aumento y escala réplicas.
- El sistema recupera rendimiento normal en menos de 1 minuto.
- CloudWatch crea métricas de CPU Burst.

---

## 4. Métricas Observadas

Se validaron:

- **Latencia promedio** antes y después del ataque.
- **Saturación de CPU/Memory** de los Pods.
- **Errores 5xx del ALB** durante fallos de DB.
- **Cantidad de réplicas** manejadas por el HPA.
- **Logs estructurados** enviados por LoggerService.
- **Reconexiones al PostgreSQL** por parte de DBService.
- **Alertas automáticas** disparadas en CloudWatch.

Todo el tránsito negativo quedó registrado en:
- CloudTrail  
- Flow Logs  
- CloudWatch Logs  

---

## 5. Conclusiones

- El sistema es **resiliente y estable** incluso bajo fallos severos.
- La API mantiene funcionamiento parcial durante pérdidas de red o latencia elevada.
- El autoscaler HPA responde correctamente frente a CPU stress.
- La arquitectura con EKS + ALB + EC2 Clone permite recuperación rápida.
- Los logs y métricas recolectados facilitan identificar la causa raíz de cualquier incidente.
- Gremlin se confirmó como una herramienta efectiva para validar disponibilidad.

---

## 6. Próximas Mejoras (Sugeridas)

- Implementar readiness/liveness probes más estrictas.
- Agregar circuit breakers e intentos automáticos con backoff exponencial.
- Evaluar migración futura de DB a Amazon RDS para mejorar RTO/RPO.
- Realizar pruebas de caos mensuales para garantizar continuidad.

