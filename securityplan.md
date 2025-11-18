# Security Plan 

## 1. IAM y Control de Acceso
- **AmazonEKSClusterRole**: permisos limitados a creación y administración del clúster.
- **AmazonEKSNodeRole**: acceso a ECR, CloudWatch y S3.
- **AmazonEC2RoleforSSM**: permite administración segura sin exponer SSH.
- **GremlinRole**: acceso restringido para ejecutar experimentos Chaos Engineering.

## 2. Cifrado y Protección de Datos
- Volúmenes EBS cifrados con KMS (AES-256).
- Snapshots DB Clone cifrados.
- Variables sensibles gestionadas con **AWS Secrets Manager**.
- Tráfico HTTPS (443) forzado en el ALB.

## 3. Auditoría y Trazabilidad
- **AWS CloudTrail** habilitado (auditoría de llamadas API).
- **VPC Flow Logs** activos.
- **CloudWatch** recopila métricas de EKS y EC2.

## 4. Seguridad de Red
- Subredes privadas para EKS y DB.
- NAT Gateways A y B para salida controlada.
- Security Groups:
  - ALB → EKS (443)
  - EKS → DB (5432)
  - Administración restringida vía SSM.

## 5. Monitoreo y Resiliencia
- CloudWatch + Grafana para métricas.
- Alarmas sobre CPU, latencia y fallos.
- Gremlin Agent instalado en EKS para pruebas de caos.
