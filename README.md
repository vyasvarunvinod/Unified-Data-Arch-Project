# Unified Accounting Data Platform

**Enterprise-grade data pipeline centralizing SAP accounting data for financial analytics and reporting**

---

## 🎯 Quick Overview

A production data architecture that extracts critical accounting datasets from **SAP ECC**, applies transformation logic, and loads into **SQL Server** using a **Medallion architecture**. Enables finance teams and analysts to perform consolidated reporting, variance analysis, and predictive modeling.

**Tech Stack:** SAP BAPI | Python | Apache Airflow | SQL Server | Docker  
**Pipeline:** Monthly automated extracts | Incremental loads | SCD Type 2 tracking  
**Impact:** 18-min extractions | 24-hour data freshness | 99.2% data accuracy

---

## 🏗️ Architecture at a Glance

```
SAP ECC (FBL5N) → Python BAPI Extraction → Medallion Architecture → Analytics Ready
                     ↓
            Bronze (Raw) → Silver (Clean) → Gold (Aggregated)
                     ↓
          Business Analysts & Data Science Teams
```

**3-Layer Medallion Pattern:**
- **Bronze:** Raw GL, AP, AR, and master data from SAP
- **Silver:** Validated, reconciled, dimensionally modeled data
- **Gold:** Fact tables and views optimized for analytics

---

## ✨ Key Highlights

✅ **SAP BAPI Integration** – Seamless extraction of GL, AP, AR modules  
✅ **Data Quality Framework** – Reconciliation, validation, anomaly detection  
✅ **Airflow Orchestration** – Monthly schedules with dependency management  
✅ **Incremental Loading** – CDC & SCD Type 2 for efficient updates  
✅ **Production-Ready** – 1h 15m end-to-end runtime, 2.3s query latency

---

## 📚 Full Documentation

For comprehensive architecture details, data models, setup instructions, and troubleshooting, see:

**[→ Complete Technical Documentation](https://github.com/vyasvarunvinod/Unified-Data-Arch-Project/blob/main/Unified_Data_Arch/Data_Arch_Design.md)**

Covers:
- Detailed layer-by-layer architecture
- Core data models (fact & dimension tables)
- Pipeline execution flow
- Data quality checks & validation rules
- Performance metrics & SLAs
- Installation & configuration
- Troubleshooting guide

---

## 💡 Why This Matters

| Problem | Solution |
|---------|----------|
| **Siloed SAP data** | Centralized, unified repository |
| **Manual exports** | Automated monthly pipeline |
| **Data quality issues** | Reconciliation & validation framework |
| **Slow reporting** | Pre-aggregated Gold layer tables |
| **No audit trail** | SCD Type 2 historical tracking |

---

## 🚀 Get Started

1. **Explore:** Check the [full documentation](https://github.com/vyasvarunvinod/Unified-Data-Arch-Project/blob/main/Unified_Data_Arch/Data_Arch_Design.md) for architecture details
2. **Setup:** Follow installation steps in the docs
3. **Deploy:** Configure SAP & SQL Server connections
4. **Monitor:** Airflow dashboard tracks monthly runs

---

## 📊 Metrics

- **Extraction Time:** 18 minutes
- **Data Freshness:** 24 hours (post SAP close)
- **Accuracy:** 99.2% (±0.01% GL reconciliation)
- **Query Response:** 2.3s average
- **Uptime:** 99.8%

---

**Built at:** Mettler-Toledo  
**Production Status:** Active (Monthly runs)  
**Last Updated:** June 2024
