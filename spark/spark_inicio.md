# pandas sería suficiente
import pandas as pd

df = pd.read_csv("pacientes_1000.csv")
sintomas_comunes = df['sintomas'].value_counts()
```

**Por qué es plano:**
- 1,000 registros → cabe en RAM
- 1 consulta simple → no hay pipeline
- 1 fuente de datos → no hay integración

---

## ✅ **ESCENARIO REAL: Donde Spark tiene sentido**

### **Caso 1: Healthcare Analytics Pipeline**

Imagina este escenario realista:
```
FUENTES DE DATOS:
├─ Hospitales (50 millones de registros históricos)
├─ Wearables (1 TB de datos de sensores por día)
├─ Laboratorios (imágenes médicas + resultados)
├─ Farmacias (prescripciones)
└─ Clima (para correlaciones ambientales)

OBJETIVO:
Predecir brotes de enfermedades respiratorias
basándose en síntomas, clima y movilidad
```

---

## PROYECTO SPARK REAL: "Sistema de Early Warning de Salud Pública"

### **Arquitectura Completa**
```
┌─────────────────────────────────────────────┐
│          INGESTA (Spark Streaming)          │
├─────────────────────────────────────────────┤
│ Kafka ← Hospital APIs (real-time)           │
│ S3 ← Wearables data (batch)                 │
│ API ← Weather services                       │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│      PROCESAMIENTO (Spark SQL + MLlib)       │
├─────────────────────────────────────────────┤
│ 1. Limpieza y normalización                 │
│ 2. Feature engineering                       │
│ 3. Agregaciones temporales                  │
│ 4. Machine Learning (clustering)            │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│           OUTPUT (Dashboards/Alertas)        │
├─────────────────────────────────────────────┤
│ PostgreSQL ← Métricas agregadas             │
│ Elasticsearch ← Búsquedas full-text         │
│ API ← Alertas en tiempo real                │
└─────────────────────────────────────────────┘