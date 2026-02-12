# 🏗️ Lakehouse Medallion Architecture — Azure Databricks + ADLS Gen2

> Pipeline end-to-end con arquitectura Bronze/Silver/Gold, ingesta multi-fuente, transformaciones con Delta Live Tables, gobierno con Unity Catalog y visualización en Power BI.

[![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)](#)
[![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=flat-square&logo=databricks&logoColor=white)](#)
[![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?style=flat-square)](#)
[![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat-square&logo=terraform&logoColor=white)](#)
[![CI/CD](https://img.shields.io/badge/Azure_DevOps-0078D7?style=flat-square&logo=azuredevops&logoColor=white)](#)

---

## 📋 Tabla de Contenidos

- [Contexto del Proyecto](#-contexto-del-proyecto)
- [Arquitectura](#-arquitectura)
- [Servicios Azure](#-servicios-azure-utilizados)
- [Flujo de Datos](#-flujo-de-datos)
- [Estructura del Repositorio](#-estructura-del-repositorio)
- [Deployment](#-deployment)
- [Costos Estimados](#-costos-estimados)
- [Métricas de Performance](#-métricas-de-performance)
- [Mejoras Futuras](#-mejoras-futuras)

---

## 🎯 Contexto del Proyecto

### Problema

Empresa del sector seguros con **10 fuentes de datos heterogéneas** (core Oracle on-premise, CRM Dynamics 365, SQL Server, APIs REST, archivos actuariales) y cero visibilidad unificada. Los reportes regulatorios tomaban **3 semanas manuales** y los datos llegaban al equipo de actuariado con **7 días de retraso**.

### Objetivo

Construir un **Data Lakehouse** centralizado sobre Azure que:
- Consolide todas las fuentes en una plataforma única gobernada
- Automatice reportes regulatorios (de 3 semanas a < 30 min)
- Proporcione datos frescos (< 1 hora) al equipo de actuariado y BI
- Implemente calidad de datos automatizada en cada capa

### Volúmenes

| Métrica | Valor |
|---------|-------|
| Datos históricos | 3 TB |
| Registros nuevos/día | ~25 millones |
| Fuentes de datos | 10 |
| Usuarios BI | 50+ |
| Retención regulatoria | 10 años |

---

## 🏛️ Arquitectura

```
┌─────────────────────────────────────────────────────────────────────┐
│                        FUENTES DE DATOS                            │
├──────────┬───────────┬──────────┬───────────┬──────────┬───────────┤
│  Oracle  │ SQL Server│ Dynamics │  APIs     │  Event   │  Archivos │
│  (Core)  │(Siniestros│  365     │  REST     │  Hubs    │  Excel    │
│          │          )│  (CRM)   │           │(Eventos) │(Actuarial)│
└────┬─────┴─────┬─────┴────┬─────┴─────┬─────┴────┬─────┴─────┬────┘
     │           │          │           │          │           │
     ▼           ▼          ▼           ▼          ▼           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    INGESTA (Azure Data Factory)                     │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐│
│  │Self-hosted│ │   Copy   │ │ Fivetran │ │ Azure    │ │   ADF    ││
│  │    IR     │ │ Activity │ │  Sync    │ │Functions │ │ SSIS IR  ││
│  │ (Oracle)  │ │ (SQL Srv)│ │(Dynamics)│ │ (APIs)   │ │ (Legacy) ││
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘│
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│              AZURE DATA LAKE STORAGE GEN2 (ADLS)                   │
│                                                                     │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐          │
│  │    BRONZE     │  │    SILVER     │  │     GOLD      │          │
│  │  (Raw Data)   │──│  (Cleaned)    │──│  (Business)   │          │
│  │  Delta Lake   │  │  Delta Lake   │  │  Delta Lake   │          │
│  │               │  │  + DLT        │  │  + Z-ordering │          │
│  │  JSON,CSV,    │  │  Expectations │  │  + Aggregates │          │
│  │  Parquet,Avro │  │  Dedup, MDM   │  │  Pre-computed │          │
│  └───────────────┘  └───────────────┘  └───────┬───────┘          │
│                                                 │                  │
└─────────────────────────────────────────────────┼──────────────────┘
                                                  │
              ┌───────────────────────────────────┤
              │                                   │
              ▼                                   ▼
┌──────────────────────┐          ┌──────────────────────────┐
│  Databricks SQL      │          │        Power BI          │
│  Warehouse (Pro)     │──────────│  Dashboards & Reportes   │
│  Photon Engine       │          │  DirectQuery + Import    │
│  Result Caching      │          │  Regulatorios SBS        │
└──────────────────────┘          └──────────────────────────┘
              │
              ▼
┌──────────────────────┐
│    Unity Catalog     │
│  Permisos por equipo │
│  Linaje automático   │
│  External Locations  │
└──────────────────────┘
```

---

## ☁️ Servicios Azure Utilizados

| Servicio | Rol | Justificación |
|----------|-----|---------------|
| **ADLS Gen2** | Almacenamiento central del Data Lake | Namespace jerárquico, ACLs granulares, lifecycle policies (Hot→Cool→Archive) |
| **Azure Data Factory** | Orquestación e ingesta | Self-hosted IR para Oracle on-premise; Copy Activity con CDC para SQL Server; parametrización para reutilizar pipelines |
| **Azure Databricks (Premium)** | Procesamiento y transformaciones | Photon Engine 5x más rápido que Spark estándar; DLT con expectations para calidad regulatoria; Unity Catalog para gobierno |
| **Delta Live Tables (DLT)** | ETL declarativo | Define expectativas de calidad en el código; recovery automático; linaje integrado |
| **Unity Catalog** | Gobierno de datos | Permisos por grupo Azure AD; linaje columna-a-columna; External Locations para ADLS |
| **Event Hubs** | Streaming de eventos | Eventos de pólizas vendidas y siniestros en near-real-time; Capture a ADLS como backup |
| **Fivetran** | Ingesta SaaS (Dynamics 365) | Sync incremental cada 15 min sin código; conector nativo certificado |
| **Power BI** | Visualización y reportes | Connected via Partner Connect al SQL Warehouse; Import mode para ejecutivos, DirectQuery para operacional |
| **Azure Key Vault** | Gestión de secretos | Service Principal credentials, connection strings; vinculado a Databricks Secret Scopes |
| **Azure DevOps** | CI/CD | YAML pipelines: lint → test → deploy staging → approval → deploy prod |
| **Terraform** | Infrastructure as Code | azurerm provider para ADLS, ADF, Databricks workspace, Key Vault, VNet |
| **Azure Monitor** | Observabilidad | Alertas de fallos en pipelines, costos por encima de threshold, degradación de rendimiento |

---

## 🔀 Flujo de Datos

### 1. Ingesta (Bronze)

```python
# ADF Copy Activity: CDC incremental desde SQL Server
# Configurado con Tumbling Window trigger cada hora
# Self-hosted IR para Oracle on-premise (no expuesto a internet)

# Event Hubs → Databricks Structured Streaming
df_stream = (spark.readStream
    .format("eventhubs")
    .options(**eh_conf)
    .load()
    .select(
        col("body").cast("string").alias("payload"),
        col("enqueuedTime").alias("event_time")
    )
)

# Auto Loader para archivos que llegan a ADLS
df_files = (spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.inferColumnTypes", "true")
    .load("abfss://bronze@storage.dfs.core.windows.net/eventos/")
)
```

### 2. Transformación (Silver)

```python
import dlt
from pyspark.sql.functions import *

@dlt.table(comment="Clientes unificados cross-system (MDM)")
@dlt.expect_or_drop("doc_no_nulo", "documento_id IS NOT NULL")
@dlt.expect_or_drop("nombre_valido", "LENGTH(TRIM(nombre)) > 2")
@dlt.expect("email_formato", "email RLIKE '^.+@.+\\\\..+$'")
def silver_clientes():
    core = dlt.read("bronze_core_clientes")
    crm = dlt.read("bronze_crm_contactos")
    siniestros = dlt.read("bronze_siniestros_asegurados")
    
    union = (core
        .unionByName(crm, allowMissingColumns=True)
        .unionByName(siniestros, allowMissingColumns=True)
        .dropDuplicates(["documento_id"])
    )
    return union
```

### 3. Agregación (Gold)

```python
@dlt.table(comment="Siniestralidad por ramo y región - mensual")
def gold_siniestralidad():
    return (
        dlt.read("silver_siniestros")
        .join(dlt.read("silver_polizas"), "poliza_id")
        .groupBy("ramo", "region", date_trunc("month", "fecha_siniestro").alias("mes"))
        .agg(
            count("*").alias("num_siniestros"),
            sum("monto_reclamado").alias("total_reclamado"),
            sum("monto_pagado").alias("total_pagado"),
            avg("dias_resolucion").alias("avg_dias_resolucion")
        )
    )
```

### 4. Optimización

```sql
-- Z-ordering en columnas más filtradas por Power BI
OPTIMIZE gold.siniestralidad ZORDER BY (region, ramo, mes);

-- Liquid Clustering para tablas con patrones de filtro cambiantes
ALTER TABLE gold.transacciones SET TBLPROPERTIES (
  'delta.enableDeletionVectors' = 'true'
);
ALTER TABLE gold.transacciones CLUSTER BY (pais, fecha);
```

---

## 📁 Estructura del Repositorio

```
01-lakehouse-medallion/
├── infra/
│   ├── main.tf                    # Terraform: ADLS, ADF, Databricks, Key Vault
│   ├── variables.tf               # Variables parametrizadas
│   ├── outputs.tf                 # Outputs (workspace URL, storage endpoints)
│   ├── environments/
│   │   ├── dev.tfvars
│   │   ├── staging.tfvars
│   │   └── prod.tfvars
│   └── modules/
│       ├── adls/                  # Módulo ADLS Gen2 con containers + lifecycle
│       ├── databricks/            # Workspace + cluster policies
│       └── adf/                   # Data Factory + linked services
│
├── adf/
│   ├── pipeline/                  # Pipelines de ADF (JSON exportado)
│   │   ├── pl_ingesta_oracle.json
│   │   ├── pl_ingesta_sqlserver.json
│   │   └── pl_master_orquestador.json
│   ├── linkedService/             # Conexiones a fuentes y destinos
│   ├── dataset/                   # Esquemas de entrada/salida
│   └── trigger/                   # Schedule + tumbling window triggers
│
├── databricks/
│   ├── src/
│   │   ├── ingesta/
│   │   │   ├── stream_event_hubs.py
│   │   │   └── autoloader_archivos.py
│   │   ├── transformaciones/
│   │   │   ├── dlt_bronze_to_silver.py
│   │   │   ├── dlt_silver_to_gold.py
│   │   │   └── mdm_clientes.py
│   │   └── calidad/
│   │       ├── expectations_silver.py
│   │       └── reconciliacion_gold.sql
│   ├── resources/
│   │   ├── workflows.yml          # Databricks Workflows definition
│   │   └── dlt_pipeline.yml       # DLT pipeline config
│   └── databricks.yml             # Databricks Asset Bundle config
│
├── tests/
│   ├── unit/
│   │   ├── test_transformaciones.py
│   │   └── test_calidad.py
│   └── integration/
│       └── test_pipeline_e2e.py
│
├── monitoring/
│   ├── alerts.bicep               # Azure Monitor alert rules
│   └── dashboard_costos.json      # Power BI dashboard de costos
│
├── docs/
│   ├── architecture.png           # Diagrama de arquitectura (draw.io)
│   ├── data-flow.md               # Documentación del flujo de datos
│   └── runbook.md                 # Runbook operativo para on-call
│
├── azure-pipelines.yml            # CI/CD con Azure DevOps
├── .pre-commit-config.yaml        # Pre-commit hooks (ruff, sqlfluff)
└── README.md                      # Este archivo
```

---

## 🚀 Deployment

### Prerrequisitos

```bash
# Azure CLI + Terraform + Databricks CLI
az login
terraform init
databricks configure --token
```

### 1. Infraestructura (Terraform)

```bash
cd infra/
terraform plan -var-file=environments/dev.tfvars
terraform apply -var-file=environments/dev.tfvars
```

### 2. Databricks Assets (DABs)

```bash
cd databricks/
databricks bundle validate -t dev
databricks bundle deploy -t dev
```

### 3. ADF Pipelines

```bash
# ADF en Git mode: merge a main → ARM template auto-generated
# Azure DevOps pipeline despliega ARM template a staging/prod
az deployment group create \
  --resource-group rg-data-prod \
  --template-file adf/ARMTemplateForFactory.json \
  --parameters adf/ARMTemplateParametersForFactory.json
```

### CI/CD Pipeline

```yaml
# azure-pipelines.yml (simplificado)
trigger:
  branches:
    include: [main]

stages:
- stage: CI
  jobs:
  - job: LintAndTest
    steps:
    - script: |
        pip install ruff pytest
        ruff check databricks/src/
        pytest tests/unit/ --junitxml=results.xml

- stage: DeployStaging
  dependsOn: CI
  jobs:
  - deployment: Staging
    environment: staging
    strategy:
      runOnce:
        deploy:
          steps:
          - script: databricks bundle deploy -t staging

- stage: DeployProd
  dependsOn: DeployStaging
  jobs:
  - deployment: Production
    environment: production  # Requires approval
    strategy:
      runOnce:
        deploy:
          steps:
          - script: databricks bundle deploy -t production
```

---

## 💰 Costos Estimados

> Basado en precios públicos de Azure (East US 2) + Databricks Premium, febrero 2025.

| Servicio | Configuración | USD/mes |
|----------|---------------|---------|
| ADLS Gen2 | 3 TB Hot + lifecycle | $55 |
| Azure Data Factory | ~200 runs/día + SSIS IR | $430 |
| Fivetran | 1 conector Standard | $200 |
| Event Hubs | Standard, 3 TU | $220 |
| Databricks Jobs Compute | i3.xlarge, Spot 70%, ~350 hrs | $700 |
| Databricks Streaming | i3.xlarge always-on, Spot | $380 |
| Databricks SQL Warehouse | Pro Medium, 1-4 scale | $1,200 |
| Power BI | 15 Pro + Premium Per User | $350 |
| Key Vault + Monitor | Secretos + alertas | $65 |
| Networking | Private endpoints, VNet | $30 |
| **TOTAL** | | **~$3,630/mes** |

### Optimizaciones aplicadas

- ✅ **Job Clusters** en producción (se destruyen al terminar): -40% vs All-Purpose
- ✅ **Spot VMs** en 70% de workers: -80% en VMs de workers
- ✅ **Photon Engine**: reduce tiempo de ejecución 3-5x = menos DBU
- ✅ **ADLS lifecycle**: Bronze > 90d → Cool, > 365d → Archive
- ✅ **SQL Warehouse auto-scale**: 1 (idle) → 4 (peak) → 1

---

## 📊 Métricas de Performance

| Métrica | Antes | Después | Mejora |
|---------|-------|---------|--------|
| Reportes regulatorios | 3 semanas manual | 25 minutos automático | **99.8% reducción** |
| Latencia datos para BI | 7 días | < 1 hora | **99.4% reducción** |
| Duplicados de clientes | 18% no detectados | 0% (MDM automático) | **100% detección** |
| Pipeline uptime | ~85% (fallos SSIS) | 99.95% | **+15 puntos** |
| Costo de auditoría | 3 semanas de prep | 4 horas (Unity Catalog) | **99.2% reducción** |
| ETL nocturno | 6 horas | 45 minutos | **87% reducción** |

---

## 🔮 Mejoras Futuras

- [ ] **Liquid Clustering** en todas las tablas Gold para eliminar particionamiento fijo
- [ ] **Delta Sharing** para compartir datos con reaseguradores sin copiar
- [ ] **Databricks Serverless SQL** para eliminar idle costs en warehouse
- [ ] **Feature Store** para alimentar modelos actuariales directamente desde Gold
- [ ] **dbt-databricks** para las transformaciones SQL con tests y documentación
- [ ] **Azure Managed Grafana** para dashboards de monitoreo técnico

---

## 📄 Licencia

Este proyecto es material de portafolio con fines demostrativos. Los datos son sintéticos y no contienen información real de ninguna empresa.
