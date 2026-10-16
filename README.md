# Migración AS400 (DB2 for i) a AWS RDS MySQL con DMS - Documentación Técnica

![AWS DMS](https://img.shields.io/badge/AWS-DMS-orange) ![AWS RDS](https://img.shields.io/badge/AWS-RDS_MySQL-blue) ![DB2](https://img.shields.io/badge/Source-AS400_DB2-green) ![Status](https://img.shields.io/badge/Status-RCA_Documentado-red)

## 📐 Arquitectura Implementada
![Arquitectura](docs/00-arquitectura.png)

> Replicación de datos desde entorno simulado AS400 a RDS MySQL usando AWS Database Migration Service. Región: us-east-2

### Componentes
- **Instancia de Replicación:** `as400-replica-vcenturion` (dms.t3.medium - 3.5.4)
- **Origen (Simulado):** AS400 DB2 for i - Librería CENTURION
- **Destino:** RDS MySQL `as400-simulado-vcenturion.cl6ieiig2g1d.us-east-2.rds.amazonaws.com:3306`

## 🎯 Objetivo
Migrar la base de datos del sistema VCenturion del AS400 a un RDS MySQL en AWS.

## 📸 Evidencia de Ejecución

### 1. Configuración de Endpoints
![Endpoints](docs/02-puntos-conexion.png)

### 2. Resultado de la Tarea de Migración
![Tarea con error](docs/01-tarea-error.png)
**Resultado:** `Carga completada con errores - 100%`
**Detalle:** Solo aparecen tablas del sistema `sys`, `mysql`, `performance_schema`. No aparecen tablas de la librería CENTURION.

## 🔍 Root Cause Analysis (RCA)
1.  El endpoint de origen se configuró como MySQL en lugar de ODBC/DB2 for i (driver IBM i Access Client Solutions requerido)
2.  No se cuenta con acceso de red/credenciales al AS400 real (IP, Puerto 8471/8476, Usuario QSECOFR, Librería)
3.  La regla de selección de tablas `%.%` trajo tablas de sistema en lugar de filtrar por librería `CENTURION.%`

## ✅ Solución Propuesta
- **Plan A (Recomendado):** Solicitar a IT acceso ODBC y recrear endpoint de origen tipo `IBM DB2 for i` con driver ODBC
- **Plan B:** Solicitar export de tablas en CSV/SAVF y hacer import con MySQL Workbench `LOAD DATA INFILE`
- **Plan C:** Usar AWS SCT (Schema Conversion Tool) para conversión de tipos DB2 -> MySQL

## 💡 Lección Aprendida
Validar siempre los Puntos de Conexión (Origen AS400 / Destino MySQL) con `Test Connection` antes de crear la tarea de migración. Un error 100% no es fallo de DMS, es error de configuración de origen.
