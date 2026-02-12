<div align="center">

# 👋 Hola, soy Jose Esteban Gallegos Aliaga

### Senior Data Engineer · Azure · Databricks · 6+ años construyendo plataformas de datos

[![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com)
[![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)](https://databricks.com)
[![Spark](https://img.shields.io/badge/Apache_Spark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)](https://spark.apache.org)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![SQL](https://img.shields.io/badge/SQL-4479A1?style=for-the-badge&logo=postgresql&logoColor=white)](#)
[![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?style=for-the-badge&logo=delta&logoColor=white)](https://delta.io)

</div>

---

## 🏗️ Sobre mí

Ingeniero de datos senior con **6+ años de experiencia** diseñando y construyendo plataformas de datos en **Microsoft Azure**. Especializado en arquitecturas **Lakehouse**, pipelines de datos a escala (batch + streaming), gobierno de datos y optimización de costos.

Mi enfoque: **plataformas production-ready** con calidad de datos automatizada, CI/CD y gobernanza desde el día uno. No solo conozco las herramientas — entiendo cuándo y por qué usar cada una para generar valor real al negocio.

```
📍 [Lima, Perú]
📧 [jgallegosa@uni.pe]
🔗 https://www.linkedin.com/in/jose-esteban-gallegos-aliaga-7a22801b4/
```

---

## 🎯 Stack Técnico

### Core Azure Data Services

| Categoría | Servicios | Nivel |
|-----------|-----------|-------|
| **Procesamiento** | Azure Databricks (Delta Lake, DLT, Unity Catalog, Photon), Synapse Analytics (Dedicated + Serverless + Spark Pools) | ⭐⭐⭐⭐⭐ |
| **Orquestación** | Azure Data Factory (pipelines complejos, triggers, parametrización, linked services, SSIS IR) | ⭐⭐⭐⭐⭐ |
| **Almacenamiento** | ADLS Gen2 (estructura medallion, ACLs, lifecycle), Blob Storage, Azure SQL Database | ⭐⭐⭐⭐⭐ |
| **Streaming** | Azure Event Hubs, Stream Analytics, Databricks Structured Streaming | ⭐⭐⭐⭐ |
| **BI & Serving** | Power BI (DirectQuery, Import, modelos semánticos), Databricks SQL Dashboards | ⭐⭐⭐⭐ |
| **Gobierno** | Microsoft Purview, Unity Catalog, Great Expectations, dbt tests | ⭐⭐⭐⭐⭐ |
| **Bases de datos** | Azure SQL Database, SQL Managed Instance, Cosmos DB (document model) | ⭐⭐⭐⭐ |

### DevOps, Seguridad & Infraestructura

| Categoría | Herramientas |
|-----------|-------------|
| **CI/CD** | Azure DevOps (YAML pipelines, Repos, Boards, Artifacts), GitHub Actions |
| **IaC** | Terraform (azurerm provider), Bicep, ARM Templates, Databricks Asset Bundles |
| **Seguridad** | Azure AD (Entra ID), Key Vault, Managed Identity, Private Endpoints, VNet injection, RBAC |
| **Monitoreo** | Azure Monitor, Application Insights, Log Analytics (KQL), Databricks System Tables |
| **Contenedores** | Azure Kubernetes Service (AKS) para Apache Airflow, Docker |
| **Code** | Python, PySpark, SQL (T-SQL, Spark SQL), Bash, dbt-databricks, dbt-synapse |

### Lenguajes & Frameworks

```text
Python/PySpark  ████████████████████████████████████  95%
SQL (T-SQL/SparkSQL) ██████████████████████████████████  90%
Terraform/Bicep  ████████████████████████████░░░░░░  80%
dbt              ████████████████████████░░░░░░░░░░  70%
KQL (Log Analytics) ██████████████████████░░░░░░░░░░  65%
Scala/Java       ████████████████░░░░░░░░░░░░░░░░░  45%
```

---


## 🚀 Proyectos Destacados

### 📂 [01 — Lakehouse Medallion Architecture](./projects/01-lakehouse-medallion/)
> **Arquitectura Lakehouse completa en Azure con Databricks + Delta Lake + ADF**

Pipeline end-to-end con capas Bronze/Silver/Gold sobre ADLS Gen2. Ingesta multi-fuente con ADF, transformaciones con Delta Live Tables, gobierno con Unity Catalog y visualización en Power BI.

| Métrica | Valor |
|---------|-------|
| Volumen procesado | 2 TB/día, 50M registros |
| Latencia E2E | < 15 minutos (near-real-time) |
| Uptime | 99.95% en 6 meses |
| Costo mensual | ~$4,200 optimizado |

**Stack:** `Azure Databricks` `ADLS Gen2` `ADF` `Delta Lake` `Unity Catalog` `Event Hubs` `Power BI` `Terraform` `Azure DevOps`

---

### 📂 [02 — Real-time Analytics Pipeline](./projects/02-realtime-analytics/)
> **Pipeline de streaming para detección de anomalías con Event Hubs + Stream Analytics + Synapse**

Procesamiento en tiempo real de transacciones financieras para detección de fraude. Arquitectura 100% Azure nativa sin Databricks, con modelo de scoring en Azure ML y dashboards en Power BI DirectQuery.

| Métrica | Valor |
|---------|-------|
| Throughput | 80M transacciones/día |
| Latencia de detección | < 30 segundos |
| Fraude prevenido (Q1) | $890,000 USD |
| Falsos positivos | 8% (vs 35% anterior) |

**Stack:** `Event Hubs` `Stream Analytics` `Synapse Dedicated` `Azure ML` `ADLS Gen2` `ADF` `SSIS` `Power BI` `Purview`

---

### 📂 [03 — Automated Data Quality Framework](./projects/03-data-quality-framework/)
> **Framework de calidad de datos con Great Expectations + DLT Expectations + Azure Monitor**

Framework reutilizable de validación de datos en múltiples capas del pipeline (Bronze, Silver, Gold). Incluye alertas automáticas, dashboard de métricas de calidad y tabla de cuarentena para registros rechazados.

| Métrica | Valor |
|---------|-------|
| Reglas de validación | 120+ expectations |
| Cobertura | 100% de tablas Gold |
| Registros rechazados detectados | ~2.3% del volumen diario |
| Tiempo de resolución de incidentes | De 48hrs a 2hrs |

**Stack:** `Great Expectations` `Databricks DLT` `Azure Monitor` `Log Analytics` `Power BI` `pytest` `Azure DevOps`

---

## 📊 Experiencia Profesional (Resumen)

```
2024 - Presente  │  Senior Data Engineer @ [Bctecnologia]
                  │  Arquitectura Lakehouse, Databricks, 
                  │  ADF, Unity Catalog, liderazgo técnico
                  │
2023 - 2024      │  Senior Data Engineer @ [Prediqt Data]
                  │  Synapse Analytics, SSIS migration,
                  │  pipelines batch + streaming
                  │
2020 - 2023      │  Data Engineer  @ [Santander Consumer]
                  │  ETL con SSIS, SQL Server, 
                  │  primeros proyectos en Azure
```


## 📫 Contacto

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/jose-esteban-gallegos-aliaga-7a22801b4/)
[![Email](https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:JGALLEGOSA@UNI.PE)
[![Portfolio](https://img.shields.io/badge/Portfolio-000000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/josegallegosa/portafolio)

</div>

---

<div align="center">
  <sub>⚡ "No solo construyo pipelines — construyo plataformas de datos confiables, gobernadas y eficientes en costo que generan valor medible para el negocio."</sub>
</div>
