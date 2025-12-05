# DeliverEasy – Laborario 3 - Arquitectura del Sistema

Este documento presenta la arquitectura del sistema **DeliverEasy**, una plataforma orientada a la gestión de órdenes y operaciones internas.  
La solución está implementada utilizando **Amazon Web Services (AWS)** y un diseño basado en contenedores y buenas prácticas de seguridad, disponibilidad y resiliencia.

El objetivo principal es ofrecer un sistema:
- Escalable
- Seguro
- Observado en tiempo real
- Tolerante a fallos
- Fácil de mantener y extender

El sistema se despliega sobre un **Cluster Amazon EKS** (Kubernetes administrado), accedido por los usuarios a través de un **Application Load Balancer**, y soportado por recursos clave como:
- **EC2 + PostgreSQL** (base de datos administrada manualmente)
- **Amazon S3** (backups cifrados)
- **Amazon CloudWatch** (monitoreo y métricas)
- **AWS KMS** (cifrado de datos)
- **AWS Secrets Manager** (manejo de credenciales)
- **AWS CloudTrail** (auditoría de eventos)
- **Gremlin Agent** (pruebas de resiliencia)

El proyecto incluye modelado visual utilizando los **4 niveles del modelo C4**:
1. **C1 – Diagrama de contexto**
2. **C2 – Diagrama de contenedores / infraestructura**
3. **C3 – Componentes internos del pod**
4. **C4 – Flujo interno detallado**

Cada uno se encuentra documentado en `architecture.md`.

## Estructura del repositorio
- **architecture.md**
- **services.md**
- **security.md**
- **mitigation.md**
- **resilience_testing.md**
- **README.md**

## Tecnologías principales

- **Backend:** Node.js en contenedores Docker
- **Orquestación:** Amazon Elastic Kubernetes Service (EKS)
- **Base de datos:** PostgreSQL (EC2)
- **Monitoreo:** CloudWatch + Grafana
- **Seguridad:** IAM, KMS, Secrets Manager
- **Auditoría:** CloudTrail
- **Resiliencia:** Gremlin Agent

---

La documentación restante se encuentra dividida en archivos especializados.
