# Security Plan

Este documento describe las medidas de seguridad implementadas en la arquitectura de DeliverEasy, siguiendo buenas prácticas de AWS Well-Architected y Kubernetes Hardening Guide.

---

## 1. IAM y Control de Acceso

### Roles principales
- **AmazonEKSClusterRole**  
  Permite que el plano de control administre los recursos necesarios del clúster.  
  Aplicamos privilegio mínimo: solo permisos requeridos para creación y operación.

- **AmazonEKSNodeRole**  
  Otorga a los nodos acceso a:
  - ECR (para descargar imágenes)
  - CloudWatch (logs y métricas)
  - S3 (backups cuando corresponde)

- **AmazonEC2RoleforSSM**  
  Permite administrar instancias sin exponer puertos SSH.  
  **Se eliminan claves públicas/privadas**, evitando vectores comunes de ataque.

- **GremlinRole**  
  Permisos estrictamente limitados solo para ejecutar experimentos de caos en los pods seleccionados.

### Buenas prácticas aplicadas
- Principio de **Least Privilege** en todas las políticas IAM.  
- **No existen usuarios IAM humanos**: todo el acceso operacional se realiza con SSO/console.  
- Rotación automática de credenciales mediante Secrets Manager.

---

## 2. Cifrado y Protección de Datos

### Cifrado en descanso
- Todos los volúmenes **EBS cifrados con AWS KMS (AES-256)**.
- Snapshots utilizados para la clonación de PostgreSQL también están cifrados.
- Los backups almacenados en **S3** están cifrados con una **Customer Managed Key (CMK)**.

### Cifrado en tránsito
- El tráfico externo pasa por el **Application Load Balancer**, forzando HTTPS (443).
- El tráfico interno entre:
  - EKS → PostgreSQL  
  - EKS → Secrets Manager  
  también utiliza TLS.

### Gestión de secretos
- Contraseñas de BD, tokens y cadenas sensibles se almacenan en **AWS Secrets Manager**.
- Los Pods obtienen los secretos en tiempo de ejecución mediante el SDK, evitando variables en texto plano.

---

## 3. Auditoría y Trazabilidad

- **AWS CloudTrail** registra todas las operaciones administrativas sobre:
  - IAM
  - EKS
  - EC2
  - S3
- **VPC Flow Logs** permiten detectar tráfico sospechoso en redes privadas.
- **CloudWatch** recopila:
  - Logs de pods (via Container Insights)
  - Logs de la base de datos
  - Métricas de CPU, memoria y errores

Todos estos servicios permiten análisis forense y cumplimiento normativo básico.

---

## 4. Seguridad de Red

### Segmentación de redes
- Arquitectura dividida en:
  - **Subredes públicas** → solo ALB, NAT e Internet Gateway.
  - **Subredes privadas** → EKS, nodos y la base de datos PostgreSQL.
- La base de datos **no es accesible desde Internet** en ningún caso.

### Security Groups
- **ALB → EKS**  
  Entrada solo por HTTPS (443).
- **EKS → PostgreSQL**  
  Solo puerto (5432), restringido al SG de los nodos.
- **Salidas controladas mediante NAT**  
  Los nodos solo salen a Internet para:
  - Pull de imágenes
  - Actualizaciones
  - Envió de logs

### NACLs (Network ACLs)
- Reglas restrictivas bloqueando tráfico no utilizado.
- Se permiten solo:
  - 443 (entrada al ALB)
  - 5432 (comunicación privada con la DB)
  - Puertos efímeros necesarios para Kubernetes

---

## 5. Kubernetes Security (RBAC, Pods y Malla interna)

- RBAC configurado para que cada servicio tenga solo permisos mínimos.
- El Pod de Orders API:
  - No corre como root.
  - Monta solamente los secretos estrictamente necesarios.
  - No expone puertos adicionales.

- NodeGroups privados (sin IP pública).
- Restricción de ingreso mediante:
  - ALB Ingress Controller
  - Security Groups
  - Policies de Kubernetes

---

## 6. Monitoreo, Alertas y Resiliencia

- **CloudWatch + Grafana** para métricas avanzadas y dashboards.
- Alarmas configuradas para:
  - CPU alta
  - Latencia anormal
  - Errores de aplicación
  - Caída de nodos

- **Gremlin Agent** instalado en el clúster para:
  - Simular fallas controladas
  - Probar resistencia del sistema
  - Validar que los mecanismos de recuperación funcionan correctamente

Las pruebas de caos están detalladas en `resilience_testing.md`.

---

## 7. Conclusión

La arquitectura implementa seguridad en todas las capas:
- Identidad
- Red
- Datos
- Código
- Contenedores
- Auditoría
- Resiliencia

Alineándose con los pilares del **AWS Well-Architected Framework** y las buenas prácticas de Kubernetes.

