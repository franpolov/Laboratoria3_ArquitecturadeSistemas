# Mitigation Plan 

## 1. Backup Original
- Se generó un **snapshot cifrado KMS** de la base de datos productiva (`snap-orders-base`).
- Backup verificado mediante restauración en EC2-Clone.

## 2. EC2 Clone
- **AMI creada:** `deliverEasyDBClone-ami`
- **Instancia:** `t3.micro`, subred privada.
- Acceso vía SSM, sin SSH público.

## 3. Restauración
- EC2-Clone restaurada correctamente desde snapshot.
- Validación de integridad DB con consultas básicas.
- Seguridad: solo EKS puede acceder al puerto 5432.

## 4. Plan de Mitigación
- En caso de fallo: lanzar nueva instancia desde AMI actualizada.
- Mantener backups incrementales cada 24 h en S3.
- Revisión mensual de permisos IAM y rotación de claves.
