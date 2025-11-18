# Mitigation Plan

Este documento detalla las acciones realizadas para garantizar la continuidad operativa del sistema DeliverEasy en caso de fallos, junto con los mecanismos de respaldo, restauración y recuperación.

---

## 1. Backup Original

- Se generó un **snapshot cifrado con AWS KMS (AES-256)** de la base de datos productiva.
- Nombre del snapshot: `snap-orders-base`.
- El snapshot fue verificado mediante su restauración en el ambiente de prueba.
- Este proceso asegura que:
  - El backup está íntegro.
  - El snapshot puede utilizarse para crear nuevas instancias funcionales.
  - El cifrado se mantiene durante todo el ciclo de vida (reposo → transporte → restauración).

---

## 2. EC2 Clone (Instancia de Recuperación)

- Se creó una **AMI personalizada y cifrada**: `deliverEasyDBClone-ami`.
- Se desplegó una instancia `t3.micro` en **subred privada**, sin exposición pública.
- Acceso únicamente mediante **AWS Systems Manager (SSM)**:
  - No existe puerto SSH abierto.
  - No se requieren claves públicas/privadas.
  - Administración más segura y auditable.
- La instancia funciona como entorno de:
  - Prueba de restauraciones.
  - Validación de integridad de datos.
  - Punto de recuperación en caso de falla crítica.

---

## 3. Restauración y Validación

- La instancia EC2-Clone fue restaurada exitosamente desde el snapshot original.
- Pruebas realizadas:
  - Conexiones SQL básicas.
  - Verificación de tablas, índices y vistas.
  - Validación de usuarios y permisos.
- Seguridad aplicada:
  - Solo el **cluster EKS** puede acceder al puerto **5432**.
  - El Security Group está configurado en modo de **acceso mínimo** (least privilege).
  - No existe tráfico entrante desde Internet gracias al aislamiento en subred privada.

---

## 4. Plan de Mitigación ante Fallos

En caso de una falla parcial o total de la base de datos principal:

### **4.1 Recuperación Rápida**
- Iniciar de inmediato una nueva instancia desde la **AMI actualizada**.
- O restaurar directamente desde el **snapshot más reciente**.
- Asociar el nuevo endpoint al cluster EKS mediante variables en **Secrets Manager**.

### **4.2 Estrategia de Backups y Retención**
- Backups automáticos e incrementales cada **24 horas** en Amazon S3.
- Ciclo de retención recomendado:
  - Diarios: 7 días.
  - Semanales: 4 semanas.
  - Mensuales: 3 meses.
- Todos los objetos S3 cifrados con la CMK asignada.

### **4.3 Mitigación Preventiva**
- Revisión mensual de:
  - Políticas IAM asociadas a EC2, EKS y S3.
  - Rotación de claves utilizadas por servicios internos.
  - Versionado y expiración de snapshots.
- Alerta en CloudWatch si:
  - La DB deja de responder.
  - La instancia presenta CPU o memoria anómala.
  - Se detecta caída del EBS root.

### **4.4 Pruebas de Confiabilidad**
- Ejecución periódica de pruebas de fallo con **Gremlin**:
  - Latencia elevada en DBService.
  - Interrupción parcial del nodo.
  - Caída simulada del proceso PostgreSQL.
- Confirmación de:
  - Comportamiento del sistema bajo estrés.
  - Tiempo de recuperación real (RTO).
  - Integridad mantenida de los datos (RPO).

---

## 5. Conclusión

El sistema cuenta con medidas robustas de respaldo, restauración y recuperación.  
La combinación de snapshots cifrados, AMIs personalizadas, aislamiento de red, gestión de secretos y pruebas de resiliencia asegura que DeliverEasy pueda mantener continuidad operativa incluso ante fallos severos.

