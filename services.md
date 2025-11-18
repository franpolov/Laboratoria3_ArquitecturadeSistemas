# Services Overview

Este documento describe todos los servicios utilizados en la arquitectura de DeliverEasy, explicando su función, la razón de uso y la relación entre ellos.

---

## 1. Amazon EKS (Elastic Kubernetes Service)

**Función:**  
Orquesta los contenedores donde se ejecuta la API principal de pedidos (Orders API).  
Proporciona escalabilidad, alta disponibilidad y separación de cargas.

**Por qué se usa:**  
- Permite desplegar microservicios de manera controlada.
- Facilita autoescalado y observabilidad.
- Se integra nativamente con IAM, CloudWatch y VPC.

**Relación con otros servicios:**
- Recibe tráfico del ALB.
- Consume secretos desde Secrets Manager.
- Se comunica con PostgreSQL (EC2 DB Clone).
- Envía logs a CloudWatch.
- Genera backups hacia S3.
- Es el host del Gremlin Agent para pruebas de caos.

---

## 2. Application Load Balancer (ALB)

**Función:**  
Punto de entrada seguro mediante HTTPS 443 para el usuario final.  
Balancea tráfico hacia los Pods del clúster EKS.

**Por qué se usa:**  
- Maneja certificados TLS.
- Soporta reglas basadas en paths/headers.
- Auto-descubrimiento con Kubernetes Ingress.

**Relación con otros servicios:**
- Usuario final → ALB → EKS.
- Integración directa con Target Groups administrados por EKS.

---

## 3. Amazon EC2 (PostgreSQL DB Clone)

**Función:**  
Servidor de base de datos PostgreSQL que almacena pedidos, usuarios y metadatos.  
Fue creado desde un snapshot cifrado para protección de datos.

**Por qué se usa:**  
- Entorno controlado para pruebas pre-producción.
- Permite restauraciones mediante snapshots.

**Relación con otros servicios:**
- Acceso exclusivo desde EKS en subred privada.
- Logs enviados hacia CloudWatch.
- Backups hacia S3.
- Acceso restringido mediante SGs.

---

## 4. Amazon S3 (Backups cifrados)

**Función:**  
Almacena snapshots y backups de la base de datos y datos relevantes.

**Por qué se usa:**  
- Alta durabilidad (11 nueves).
- Integración con KMS para cifrado de objetos.
- Es económico y escalable.

**Relación con otros servicios:**
- PostgreSQL envía backups programados.
- EKS puede almacenar artefactos o registros temporales.

---

## 5. AWS Secrets Manager

**Función:**  
Gestión segura de credenciales y secretos (passwords de DB, tokens, etc.).

**Por qué se usa:**  
- Rotación automática.
- No se almacenan secretos en variables de entorno.
- Integración directa con AWS SDK desde los Pods.

**Relación con otros servicios:**
- Orders API obtiene credenciales en runtime.
- IAM controla permisos de lectura a nivel de Pod.

---

## 6. AWS KMS (Key Management Service)

**Función:**  
Cifrado de datos en reposo: EBS, S3 y snapshots.

**Por qué se usa:**  
- Protege claves sensibles.
- Permite auditoría de uso y control granular.
- Requisito esencial para entornos seguros.

**Relación con otros servicios:**
- Cifra volúmenes EBS del EC2 y los nodos EKS.
- Cifra backups en S3.
- Cifra todos los snapshots del clúster.

---

## 7. Amazon CloudWatch (Logs y métricas)

**Función:**  
Recolecta logs, métricas y alarmas del sistema.

**Por qué se usa:**  
- Paneles en Grafana.
- Analítica en tiempo real.
- Alarmas de salud de la aplicación.

**Relación con otros servicios:**
- EKS → Logs de Pods
- PostgreSQL → Logs de base de datos
- Gremlin → Evidencia de resiliencia
- NAT Gateways / ALB → métricas de red

---

## 8. AWS CloudTrail (Auditoría)

**Función:**  
Registra todas las llamadas API para operaciones administrativas.

**Por qué se usa:**  
- Auditoría regulatoria.
- Detección de accesos no autorizados.
- Análisis forense si ocurre un incidente.

**Relación con otros servicios:**
- Monitorea IAM, EC2, EKS, S3 y más.
- Se complementa con Flow Logs y CloudWatch.

---

## 9. VPC (Virtual Private Cloud)

**Función:**  
Aisla la red del proyecto en subredes públicas y privadas.

**Por qué se usa:**  
- Segregación de capas (ALB público → EKS → DB).
- Control total del tráfico interno.
- Permite zero-trust network policies.

### Componentes:
- **Subredes públicas:** ALB, NAT y gateway de internet.
- **Subredes privadas:** EKS y PostgreSQL.
- **NAT Gateways:** salida controlada para nodos privados.
- **Security Groups:** reglas específicas de entrada/salida.
- **Network ACLs:** segunda capa de filtrado.

---

## 10. Gremlin Agent (Chaos Engineering)

**Función:**  
Simula fallos para probar resiliencia del sistema.

**Por qué se usa:**  
- Validar tolerancia a fallos.
- Evaluar recuperación automática.
- Probar autoescalado y comportamiento bajo estrés.

**Relación con otros servicios:**
- Ejecuta experimentos sobre Pods en EKS.
- Registra eventos en CloudWatch.
- Interactúa con DBService y LoggerService durante fallos.

---

# Conclusión

La arquitectura combina servicios totalmente administrados (EKS, ALB, CloudWatch) con componentes críticos (PostgreSQL en EC2) y servicios de seguridad (KMS, Secrets Manager) para entregar una solución completa, segura y resiliente para el proyecto DeliverEasy.

