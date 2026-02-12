# ⚡ Real-time Analytics Pipeline — Detección de Fraude Financiero

> Pipeline de streaming para detección de anomalías en transacciones bancarias con Event Hubs + Stream Analytics + Synapse + Azure ML. Arquitectura 100% Azure nativa.

[![Azure](https://img.shields.io/badge/Azure-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)](#)
[![Synapse](https://img.shields.io/badge/Synapse-0078D4?style=flat-square)](#)
[![Stream Analytics](https://img.shields.io/badge/Stream_Analytics-0078D4?style=flat-square)](#)
[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=flat-square&logo=powerbi&logoColor=black)](#)

---

## 🎯 Contexto del Proyecto

### Problema

Banco comercial con **3M clientes activos** y **80M transacciones diarias**. El sistema de detección de fraude existente usaba reglas manuales con **35% de falsos positivos**, causando bloqueos innecesarios de tarjetas y quejas de clientes. El Data Warehouse on-premise (SQL Server, 12 TB) estaba al límite: el ETL nocturno tardaba **9 horas** y superaba la ventana operativa.

### Restricción clave

El banco decidió **no usar Databricks** por política de vendor lock-in: arquitectura 100% Azure nativa con soporte unificado Microsoft y Enterprise Agreement existente.

### Objetivo

- Detección de fraude en **< 30 segundos** (vs horas con el sistema anterior)
- ETL nocturno en **< 3 horas** (vs 9 horas)
- Migración de **47 paquetes SSIS** al cloud sin reescribir
- Dashboard de fraude en **tiempo real** para el equipo de operaciones

---

## 🏛️ Arquitectura

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         FUENTES EN TIEMPO REAL                          │
├──────────────┬──────────────┬──────────────┬─────────────┬──────────────┤
│  POS/ATM     │  Banca       │  Tarjetas    │  Interbanco │  Canales     │
│  Transacc.   │  Móvil       │  Crédito     │  SWIFT/ACH  │  Digitales   │
└──────┬───────┴──────┬───────┴──────┬───────┴──────┬──────┴──────┬───────┘
       │              │              │              │             │
       └──────────────┴──────────────┴──────┬───────┴─────────────┘
                                            │
                                            ▼
                               ┌────────────────────────┐
                               │   Azure Event Hubs     │
                               │   (Dedicated, 6 TU)    │
                               │   14 días retención     │
                               └─────────┬──────────────┘
                                         │
                          ┌──────────────┼──────────────┐
                          │              │              │
                          ▼              ▼              ▼
               ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐
               │   Stream     │ │  Event Hubs  │ │   Databricks     │
               │  Analytics   │ │   Capture    │ │   (future opt.)  │
               │  (Anomaly    │ │  → ADLS      │ │                  │
               │   Detection) │ │   Bronze     │ │                  │
               └──────┬───────┘ └──────┬───────┘ └──────────────────┘
                      │                │
                      ▼                ▼
           ┌──────────────┐   ┌──────────────────────────────────────┐
           │   Synapse    │   │        ADLS Gen2                     │
           │  Dedicated   │   │  ┌────────┐ ┌────────┐ ┌────────┐  │
           │  (Alertas    │   │  │ Bronze │→│ Silver │→│  Gold  │  │
           │   de Fraude) │   │  │Parquet │ │ Delta  │ │ Delta  │  │
           └──────┬───────┘   │  └────────┘ └────────┘ └───┬────┘  │
                  │           └─────────────────────────────┼───────┘
                  │                                         │
                  ▼                                         ▼
           ┌──────────────┐                    ┌──────────────────┐
           │  Power BI    │                    │  Synapse         │
           │  DirectQuery │                    │  Serverless SQL  │
           │  (Dashboard  │                    │  (Ad-hoc queries)│
           │   Real-time) │                    └──────────────────┘
           └──────────────┘
```

---

## ☁️ Servicios Azure Utilizados

| Servicio | Rol | Justificación |
|----------|-----|---------------|
| **Event Hubs Dedicated** | Ingesta streaming | 80M txn/día requiere throughput dedicado; compatible con Kafka; 14d retención para replay |
| **Stream Analytics** | Detección de anomalías en SQL | Tumbling windows de 5 min; anomaly detection nativo; output a Synapse y ADLS |
| **Synapse Dedicated SQL Pool** | Data Warehouse central | DW500c con HASH distribution; columnstore indexes; result set caching |
| **Synapse Spark Pools** | Transformaciones pesadas | PySpark para Bronze→Silver→Gold; auto-pause para ahorrar cuando idle |
| **Synapse Serverless SQL** | Consultas ad-hoc | Pay-per-TB para exploración sin infraestructura dedicada |
| **ADLS Gen2** | Data Lake (Medallion) | Bronze (Parquet), Silver/Gold (Delta via Synapse Spark) |
| **Azure ML** | Modelo de scoring de fraude | XGBoost entrenado en datos históricos; managed endpoint para batch scoring |
| **SSIS (Azure-SSIS IR)** | Migración de ETL legacy | 47 paquetes migrados sin cambios; modernización gradual |
| **ADF** | Orquestación batch | Copy Activity, Tumbling Window triggers, REST connectors |
| **Purview** | Gobernanza y clasificación PII | Scans automáticos; clasificación de 23 columnas PII; data masking |
| **Power BI Premium** | Dashboards operativos y regulatorios | DirectQuery para real-time; scheduled refresh para reportes |

---

## 🔀 Flujo de Datos

### Streaming Path (Fraude)

```sql
-- Stream Analytics: detección multi-señal
SELECT
    System.Timestamp() AS window_end,
    tarjeta_id,
    COUNT(*) AS num_txn,
    SUM(monto_usd) AS total_monto,
    COUNT(DISTINCT pais) AS paises_distintos,
    COLLECT(pais) AS lista_paises
FROM TransaccionesInput TIMESTAMP BY evento_timestamp
GROUP BY tarjeta_id, TumblingWindow(minute, 5)
HAVING
    COUNT(DISTINCT pais) > 2           -- Multi-país en 5 min
    OR COUNT(*) > 5                     -- Velocidad alta
    OR SUM(monto_usd) > 5000           -- Monto acumulado alto
```

### Batch Path (ETL Nocturno)

```python
# Synapse Spark: Bronze → Silver con validaciones
from pyspark.sql.functions import *

df_bronze = spark.read.parquet(
    "abfss://bronze@storage.dfs.core.windows.net/transacciones/"
)

df_silver = (df_bronze
    .filter("cuenta_id IS NOT NULL AND monto > 0")
    .dropDuplicates(["transaccion_id"])
    .withColumn("fecha", to_date("timestamp"))
    .withColumn("hora", hour("timestamp"))
)

df_silver.write.format("delta").mode("append") \
    .partitionBy("fecha") \
    .save("abfss://silver@storage.dfs.core.windows.net/transacciones/")
```

### Synapse Dedicated: Star Schema

```sql
-- Fact table: HASH distribution + Columnstore
CREATE TABLE gold.fact_transacciones
WITH (
    DISTRIBUTION = HASH(cuenta_id),
    CLUSTERED COLUMNSTORE INDEX,
    PARTITION (fecha RANGE RIGHT FOR VALUES (
        '2024-01-01','2024-04-01','2024-07-01','2024-10-01'
    ))
) AS
SELECT
    t.transaccion_id,
    t.cuenta_id,
    t.sucursal_id,
    t.tipo_transaccion,
    t.monto,
    t.fecha,
    t.hora,
    COALESCE(f.fraud_score, 0) AS fraud_score,
    f.is_fraud_alert
FROM silver.transacciones t
LEFT JOIN gold.fraud_scores f ON t.transaccion_id = f.transaccion_id;

-- Dimensión replicada (elimina shuffles en JOINs)
CREATE TABLE gold.dim_sucursal
WITH ( DISTRIBUTION = REPLICATE )
AS SELECT * FROM staging.sucursales;
```

---

## 📁 Estructura del Repositorio

```
02-realtime-analytics/
├── infra/
│   ├── main.bicep                 # Bicep: Event Hubs, Synapse, ADLS, Stream Analytics
│   ├── parameters.dev.json
│   └── parameters.prod.json
│
├── streaming/
│   ├── stream-analytics/
│   │   ├── fraud_detection.asaql  # Query de detección de fraude
│   │   └── config.json            # Input/Output bindings
│   └── event-hubs/
│       └── producer_simulator.py  # Simulador de transacciones para testing
│
├── synapse/
│   ├── notebooks/
│   │   ├── bronze_to_silver.py    # Synapse Spark: limpieza y validación
│   │   └── silver_to_gold.py      # Agregaciones y star schema
│   ├── sql-scripts/
│   │   ├── create_tables.sql      # DDL: fact + dimensions con distribución
│   │   ├── scd_type2.sql          # Slowly Changing Dimensions
│   │   └── reconciliation.sql     # Validaciones post-carga
│   └── pipelines/
│       └── pl_etl_nocturno.json   # Synapse Pipeline (fork ADF)
│
├── ml/
│   ├── train_fraud_model.py       # XGBoost training pipeline
│   ├── score_batch.py             # Batch scoring diario
│   └── model_config.yml           # Hyperparámetros y features
│
├── ssis/
│   ├── packages/                  # 47 paquetes .dtsx migrados
│   └── migration_notes.md         # Documentación de migración
│
├── monitoring/
│   ├── kql_queries/
│   │   ├── pipeline_failures.kql  # Log Analytics: fallos de pipeline
│   │   └── cost_tracking.kql      # Seguimiento de costos diario
│   └── alerts.bicep               # Reglas de alerta
│
├── tests/
│   ├── test_stream_analytics.py   # Tests de queries SA
│   └── test_etl.py                # Tests de transformaciones
│
├── azure-pipelines.yml            # CI/CD
└── README.md
```

---

## 💰 Costos Estimados

| Servicio | Configuración | USD/mes |
|----------|---------------|---------|
| ADLS Gen2 | 12 TB (Hot + lifecycle) | $180 |
| Event Hubs Dedicated | 6 TU, 14d retención | $750 |
| Stream Analytics | 6 SU always-on | $520 |
| Synapse Dedicated | DW500c Reserved 1yr | $1,800 |
| Synapse Spark Pool | Medium, ~300 hrs, auto-pause | $550 |
| Synapse Serverless | ~3 TB scanned | $150 |
| ADF + SSIS IR | Orchestration + 47 paquetes | $1,200 |
| Azure ML | Compute + endpoint | $420 |
| Purview | Scans + clasificación | $280 |
| Power BI Premium PU | 30 usuarios | $600 |
| Key Vault + Monitor | Enterprise monitoring | $170 |
| **TOTAL** | | **~$6,620/mes** |

---

## 📊 Métricas de Performance

| Métrica | Antes | Después | Mejora |
|---------|-------|---------|--------|
| ETL nocturno | 9 horas (fallos frecuentes) | 2h 15min, zero fallos | **75% más rápido** |
| Detección de fraude | Horas (batch) | < 30 segundos | **99.9% más rápido** |
| Falsos positivos | 35% | 8% | **-77%** |
| Fraude prevenido (Q1) | ~$200K (estimado) | $890,000 | **4.5x mejora** |
| Query time (top 10) | 45 minutos | 12 segundos | **99.6% reducción** |
| Paquetes SSIS migrados | 0/47 | 47/47 (3 semanas) | **100%** |

---

## 🔮 Mejoras Futuras

- [ ] **Migrar a Databricks** cuando se levante la restricción de vendor (Photon + DLT)
- [ ] **Azure Cosmos DB** para scoring de fraude en real-time (< 10ms)
- [ ] **dbt-synapse** para reemplazar stored procedures con modelos SQL versionados
- [ ] **Modernizar SSIS** restante (32/47 pendientes) a ADF Mapping Data Flows
- [ ] **Managed Grafana** para dashboards técnicos de operaciones
- [ ] **Synapse Link** para Cosmos DB → Synapse analytical queries sin ETL

---

## 📄 Licencia

Proyecto demostrativo con datos sintéticos. No contiene información real de ninguna entidad financiera.
