# Unified Accounting Data Platform

A scalable, enterprise-grade data pipeline that centralizes accounting data from SAP ECC across multiple business units into a unified repository for finance and analytics teams.

## Overview

This project automates the extraction of critical accounting datasets from SAP ECC, applies standardized transformation logic, and loads them into a centralized SQL Server data warehouse using a Medallion architecture pattern. The platform enables business analysts and data science teams to perform consolidated financial reporting, variance analysis, and predictive modeling.

## Architecture

```
┌─────────────────────┐
│   Source Layer      │
│  SAP ECC - FBL5N    │
│  (Tcode: FBL5N)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────────────────┐
│      Ingestion Layer                        │
│  On-Prem Server                             │
│  ├─ Docker Containerization                 │
│  └─ Python SAP BAPI Connector               │
└──────────┬──────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│      Medallion Architecture (SQL Server)                │
│                                                         │
│  ┌──────────┐   ┌─────────────┐   ┌─────────┐        │
│  │ Extract  │──▶│Transformation│──▶│  Load   │        │
│  │ (Bronze) │   │   (Silver)   │   │ (Gold)  │        │
│  └──────────┘   └─────────────┘   └─────────┘        │
│                                                         │
│  • Raw GL Data  • Validations    • Cleaned GL        │
│  • AP/AR Data   • Reconciliation  • AP/AR Views       │
│  • Cost Centers • SCD Type 2      • CC Hierarchies    │
│  • Masters      • Aggregations    • Fact Tables       │
└──────────┬──────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────┐
│     End-Users                    │
│  • Business Analysts             │
│  • Data Science Team             │
│  • Finance & Reporting           │
└──────────────────────────────────┘
```

### Architecture Layers

#### **Bronze Layer** (Raw Extraction)
- Direct dumps from SAP ECC tables (GL, AP, AR, Cost Centers)
- Minimal transformation; retains source data fidelity
- Partitioned by extraction date for incremental loading

#### **Silver Layer** (Transformation & Cleansing)
- Data validation and quality checks (null handling, duplicates)
- SAP code-to-business mapping (GL account hierarchies, cost center dimensions)
- Slowly Changing Dimension (SCD Type 2) for historical tracking
- Reconciliation checks against source tables
- Calculated fields (cost allocations, exchange rate adjustments)

#### **Gold Layer** (Business-Ready)
- Aggregated fact tables (GL balances, AP aging, AR collections)
- Denormalized dimension tables (Accounts, Cost Centers, Profit Centers)
- Pre-built views for common analytics use cases
- Optimized indexing for query performance

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| **Source System** | SAP ECC (FBL5N Transaction) |
| **Data Extraction** | Python, SAP BAPI Connector |
| **Orchestration** | Apache Airflow |
| **Data Warehouse** | SQL Server |
| **Containerization** | Docker |
| **Infrastructure** | On-Premises Server |

---

## Key Features

### 1. **SAP BAPI Integration**
- Seamless connection to SAP ECC via BAPI calls
- Extracts GL (General Ledger), AP (Accounts Payable), and AR (Accounts Receivable) modules
- Handles large data volumes with batching strategies
- Retry logic for transient failures

### 2. **Medallion Architecture**
- **Bronze → Silver → Gold** data flow ensures data quality and reusability
- Separation of concerns for maintainability and scalability
- Support for incremental loads and full refreshes

### 3. **Data Quality Framework**
- **Row-count validation**: Verify record counts match source extracts
- **Null checks**: Flag incomplete records for investigation
- **Reconciliation logic**: Balance GL totals against SAP source tables
- **Data profiling**: Anomaly detection for unexpected value distributions

### 4. **Orchestration & Scheduling**
- **Apache Airflow DAGs** manage pipeline execution
- Monthly scheduled runs with dependency management
- Ad-hoc load capability for urgent data requests
- Automated failure alerts and retry mechanisms

### 5. **Incremental Loading**
- Change Data Capture (CDC) for efficient updates
- SCD Type 2 tracking historical changes to master data
- Avoids full reloads; only processes modified records
- Optimized SQL merge operations for upserts

---

## Data Models

### Core Tables

#### **fact_gl_balances** (Gold Layer)
```
- period_date
- gl_account_code
- cost_center_code
- debit_amount
- credit_amount
- balance_amount
- company_code
- fiscal_year
- effective_date
- end_date (SCD Type 2)
```

#### **fact_ap_transactions** (Gold Layer)
```
- invoice_date
- vendor_code
- invoice_number
- line_item_amount
- tax_amount
- due_date
- payment_date (nullable)
- company_code
```

#### **dim_gl_accounts** (Gold Layer)
```
- gl_account_code (PK)
- account_description
- account_type
- cost_element_type
- profit_center
- effective_date
- end_date (SCD Type 2)
```

#### **dim_cost_centers** (Gold Layer)
```
- cost_center_code (PK)
- cost_center_name
- department
- profit_center_code
- manager_id
- active_flag
- effective_date
- end_date (SCD Type 2)
```

---

## Pipeline Execution Flow

### Monthly Pipeline

1. **Extraction Phase** (T+0)
   - Trigger SAP BAPI calls for GL, AP, AR, and master data
   - Load raw data into Bronze layer tables
   - Log extraction timestamps and record counts

2. **Transformation Phase** (T+0 to T+2)
   - Validate data completeness and correctness
   - Apply business logic mappings and calculations
   - SCD Type 2 updates for dimension tables
   - Load cleaned data into Silver layer

3. **Loading Phase** (T+2)
   - Aggregate transactions into fact tables
   - Perform reconciliation against source GL balances
   - Load final datasets into Gold layer
   - Generate data quality reports

4. **Monitoring Phase** (Continuous)
   - Alert teams on validation failures
   - Track pipeline runtime metrics
   - Archive logs for audit compliance

### Trigger Mechanisms
- **Scheduled**: Runs on the 5th of each month at 02:00 UTC
- **Manual**: Business teams can trigger ad-hoc loads via Airflow UI
- **Event-based**: Triggered on SAP month-end close completion

---

## Setup & Installation

### Prerequisites
- Python 3.8+
- Apache Airflow 2.x
- SQL Server 2019+
- Docker (for containerization)
- SAP RFC SDK

### Installation

```bash
# Clone repository
git clone https://github.com/your-repo/unified-accounting-data-platform.git
cd unified-accounting-data-platform

# Install Python dependencies
pip install -r requirements.txt

# Configure SAP BAPI credentials
export SAP_HOST="<sap-host>"
export SAP_CLIENT="<sap-client>"
export SAP_USER="<sap-user>"
export SAP_PASSWORD="<sap-password>"

# Configure database connections
export SQLSERVER_HOST="<sql-server-host>"
export SQLSERVER_DB="<database-name>"
export SQLSERVER_USER="<user>"
export SQLSERVER_PASSWORD="<password>"

# Build Docker image
docker build -t accounting-data-platform:latest .

# Start Airflow services
airflow db init
airflow webserver --port 8080
airflow scheduler
```

### Configuration Files

**config/sap_config.yaml**
```yaml
sap:
  host: ${SAP_HOST}
  client: ${SAP_CLIENT}
  user: ${SAP_USER}
  password: ${SAP_PASSWORD}

extractions:
  - table: BKPF
    description: "GL Header"
  - table: BSEG
    description: "GL Line Items"
  - table: BSAK
    description: "AP Invoice Headers"
  - table: BSCK
    description: "AR Invoice Headers"
```

**config/database_config.yaml**
```yaml
sqlserver:
  host: ${SQLSERVER_HOST}
  database: ${SQLSERVER_DB}
  user: ${SQLSERVER_USER}
  password: ${SQLSERVER_PASSWORD}
  
layers:
  bronze: "bronze_schemas"
  silver: "silver_schemas"
  gold: "gold_schemas"
```

---

## Data Quality Checks

### Validation Rules

| Check | Description | Threshold |
|-------|-------------|-----------|
| **Row Count Match** | Bronze ≥ 95% of SAP source | 95% |
| **Null Validation** | No nulls in key fields (GL Account, Cost Center) | 0% tolerance |
| **Balance Reconciliation** | Gold GL total = SAP FBL5N report balance | ±0.01% |
| **Duplicate Detection** | No duplicate transactions in Silver/Gold | 0 duplicates |
| **Date Validity** | Document dates within fiscal period | 100% valid |

### Alerting

- **Critical**: Pipeline failure, reconciliation mismatch → Immediate Slack/Email
- **Warning**: >5% row count variance → Team lead notification
- **Info**: Successful execution logs → Dashboard dashboard

---

## Performance Metrics

| Metric | Target | Current |
|--------|--------|---------|
| **Extraction Time** | < 30 minutes | 18 min |
| **Transformation Time** | < 45 minutes | 32 min |
| **Total Pipeline Runtime** | < 2 hours | 1h 15m |
| **Data Freshness** | ≤ 48 hours post SAP close | 24h |
| **Query Response (Gold)** | < 5 seconds | 2.3s avg |

---

## Known Limitations & Future Work

### Current Limitations
- On-premises deployment limits scalability; cloud migration pending
- Monthly batching may not suit real-time analytics use cases
- SCD Type 2 can increase storage overhead for high-cardinality dimensions

### Roadmap
- [ ] Migrate to cloud (Azure Data Lake / Databricks)
- [ ] Implement CDC for near-real-time updates
- [ ] Add predictive models for cash flow forecasting
- [ ] Expand to include cost accounting (COKP, COEP)
- [ ] Integrate with Power BI for automated reporting

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m "Add feature"`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request with detailed description

### Code Standards
- PEP 8 compliance for Python
- SQL formatting per project conventions
- Unit tests for new extraction/transformation logic
- Documentation for all DAGs and functions

---

## Troubleshooting

### Common Issues

#### Issue: "SAP Connection Timeout"
**Solution**: Verify RFC SDK is installed; check SAP_HOST and network connectivity
```bash
python -c "from pyrfc import Client; print('RFC OK')"
```

#### Issue: "SQL Server Authentication Failed"
**Solution**: Verify credentials and SQLSERVER_HOST is reachable
```bash
sqlcmd -S <sqlserver-host> -U <user> -P <password> -Q "SELECT 1"
```

#### Issue: "Airflow DAG not triggering"
**Solution**: Check scheduler is running and DAG is in active state
```bash
airflow dags list
airflow scheduler restart
```

---

## Support & Contact

For questions or issues, please:
- Open a GitHub Issue with detailed logs
- Contact the data engineering team: [data-eng@company.com]
- Check internal wiki: [Wiki Link]

---

## License

This project is proprietary and internal to Mettler-Toledo. Unauthorized access or distribution is prohibited.

---

## Changelog

### v1.0.0 (2024)
- Initial release with Medallion architecture
- SAP BAPI integration (GL, AP, AR modules)
- Airflow orchestration and scheduling
- Data quality framework

### v1.1.0 (Planned)
- Cost accounting module integration
- Near-real-time data pipeline
- Power BI integration

---

**Last Updated**: June 2024  
**Maintained By**: Data Engineering Team  
**Repository**: [Link to GitHub Repo]