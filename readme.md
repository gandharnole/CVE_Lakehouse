# 🧠 CVE Lakehouse on Databricks  
### Course: DIC 587 – Data Intensive Computing | Fall 2025  
**Author:** Gandhar Sidhaye  
**Instructor:** —  
**Platform:** Databricks Community Edition (Free Tier)  
**Runtime:** DBR 13.x (Spark 3.x, Python 3)

---

## 📘 Project Overview

This project implements a **cybersecurity vulnerability intelligence platform** using real-world Common Vulnerabilities and Exposures (CVE) data from the [CVEProject/cvelistV5](https://github.com/CVEProject/cvelistV5) repository.  
The goal is to design a **three-layer Medallion Architecture** (Bronze → Silver → Gold) that ingests, normalizes, and analyzes vulnerability data at scale using **Databricks and Delta Lake** principles.

It simulates a real-world **Data Lakehouse pipeline** that transforms unstructured JSON vulnerability feeds into structured, queryable analytics tables capable of producing **risk intelligence dashboards** for cybersecurity teams.

---

## 🏗️ Architecture Overview

### 🔹 Medallion Architecture Stages

| Layer | Description | Output |
|--------|-------------|---------|
| **Bronze** | Ingests raw JSON CVE records directly from GitHub (CVE v5 format). Preserves full metadata and structure. | `df_2024` |
| **Silver** | Normalizes and flattens semi-structured JSON data into relational tables. Implements the **explode** operation to extract vendor–product relationships. | `df_core`, `df_aff` |
| **Gold** | Performs business-ready exploratory data analysis (EDA) — severity distribution, top vendors, monthly disclosure trends, and vendor risk profiling. | Aggregated DataFrames + visualizations |


---

## 📂 Dataset Description

### Source
**Repository:** [CVEProject/cvelistV5](https://github.com/CVEProject/cvelistV5)  
**Format:** JSON v5 schema  
**Data Scope:** CVEs published in **2024** (or fallback to 2023 if unavailable)

### Structure Example
Each record (e.g., `CVE-2024-12345.json`) contains:
- **Metadata:** CVE ID, publication & update timestamps, state, and assigner.  
- **Containers:** Detailed fields like affected vendor/product, versions, CVSS scores, and descriptions.

### Data Volume
- ~40,000+ CVE records globally (filtered sample of 3,000–5,000 processed for Databricks CE).  
- ~1 GB compressed repository size.

---

## 🧩 Implementation Details

### 1️⃣ Bronze Layer – Raw Ingestion
- Downloads the entire `cvelistV5` repository as a ZIP directly from GitHub.  
- Extracts only CVE JSONs for the year 2024 using recursive file lookup.  
- Parses and flattens JSON fields (`cveMetadata`, `containers.cna`) into a simplified schema.  
- Filters by `cveMetadata.datePublished` to isolate 2024 vulnerabilities.  
- Creates the **Bronze table (`df_2024`)** as the immutable source of truth.

**Key Learnings:**
- Handling nested JSON with Spark.  
- Schema-on-read approach.  
- Recursive ingestion pattern for hierarchical repositories.  

---

### 2️⃣ Silver Layer – Normalization
- Builds two relational tables:
  - **Core Table (`df_core`):**  
    One row per CVE with metadata, publication date, and CVSS v3.0/v3.1 scores.
  - **Affected Table (`df_aff`):**  
    One row per vendor–product pair, derived by `explode_outer()` of nested arrays.
- Applies timestamp formatting, null handling, and type standardization.

**Key Learnings:**
- Use of Spark SQL and PySpark for data normalization.  
- Data integrity validation (unique CVE IDs, non-null keys).  
- Conversion from semi-structured to relational schema.  

---

### 3️⃣ Gold Layer – Exploratory Data Analysis (EDA)
The Gold layer transforms structured Silver data into business-ready cybersecurity intelligence.

#### Performed Analyses:
| Analysis | Description | Output |
|-----------|--------------|---------|
| **Severity Distribution** | Groups CVEs by CVSS score bucket (Critical/High/Medium/Low). | Bar chart + counts |
| **Top Vendors** | Identifies vendors with highest number of reported CVEs. | Top 15 table |
| **Monthly Trend** | Tracks publication frequency across months to detect seasonal spikes. | Time-series chart |
| **Vendor Risk Profile** | Combines CVE counts and average CVSS to rank high-risk vendors. | Risk index table |

#### Key Findings:
- Most vulnerabilities were rated *Medium* or *High* severity.  
- Top vendors are large open-source ecosystems with extensive dependency trees.  
- Peak disclosures occurred mid-year, consistent with coordinated vulnerability reporting cycles.  
- Critical vulnerabilities cluster around popular server and software frameworks.

---

## 🧮 Technologies Used

| Category | Tools / Libraries |
|-----------|------------------|
| **Platform** | Databricks Community Edition (Free Tier) |
| **Engine** | Apache Spark (PySpark API) |
| **Language** | Python 3 |
| **Data Format** | JSON (v5 schema), in-memory Delta-like DataFrames |
| **Visualization** | Matplotlib, Pandas |
| **Storage** | In-memory (no DBFS/UC for CE compatibility) |

---

## 🧰 How to Run This Project

### Step 1 – Setup
1. Log into [Databricks Community Edition](https://community.cloud.databricks.com/).  
2. Create a new **cluster** using DBR 13.x (or higher).  
3. Create three notebooks:
   - `01_ingest_cvelist_inmemory` → Bronze  
   - `02_bronze_to_silver_inmemory` → Silver  
   - `03_gold_layer_analysis` → Gold  

### Step 2 – Run in Sequence
1. Execute the **Bronze** notebook to download and load the dataset.  
2. Run the **Silver** notebook to normalize the data into `df_core` and `df_aff`.  
3. Finally, open the **Gold** notebook and perform EDA visualizations.

⚠️ **Note:** Keep the cluster running between notebooks — variables are in-memory only.

---

## 📈 Key Outputs & Screenshots

| Output | Description |
|---------|-------------|
| ✅ `df_2024.count()` | Verifies 3k–5k 2024 CVEs loaded |
| ✅ `df_core.show()` | Core vulnerability table preview |
| ✅ `df_aff.show()` | Vendor–product exploded view |
| 📊 Severity Chart | Distribution of vulnerabilities by severity |
| 📈 Monthly Trend | CVE publication trend across months |
| 🧾 Risk Table | Top vendors ranked by average CVSS & critical count |

---

## 🧠 Findings & Conclusion

- **Majority of vulnerabilities** fall within the *Medium–High* severity range, requiring consistent but manageable patching schedules.  
- **Critical vulnerabilities** (CVSS ≥ 9) form a small but impactful portion, demanding immediate attention.  
- **Vendor concentration analysis** reveals that top software ecosystems accumulate a large share of vulnerabilities due to code reuse and dependency exposure.  
- **Monthly trends** highlight spikes aligning with major security events and patch cycles.  

### Final Takeaway
The **CVE Lakehouse** demonstrates how **data engineering + analytics** can extract real-time security intelligence from public data sources.  
By implementing the **Medallion Architecture** pattern, the project achieves:
- Traceable data lineage (Bronze)  
- Normalized schema for analysis (Silver)  
- Business insights with contextual intelligence (Gold)  

This pipeline provides a scalable foundation for future work such as integrating the NVD API, KEV catalog, or EPSS probability scoring to enhance predictive vulnerability management.

---

## 🧩 Future Enhancements
- Stream ingestion of CVE data via **CVE Services API** (real-time Bronze updates).  
- Integration with **MITRE CWE** for weakness classification.  
- **XGBoost / ML-based severity prediction** to identify potential zero-days.  
- **Interactive dashboard** using Streamlit or Power BI.

---

## 🏁 Acknowledgments
This project was completed as part of **DIC 587 – Data Intensive Computing** at the **University at Buffalo (UB)**.  
Special thanks to the CVE Project, NVD, and the Databricks Community for making open vulnerability data accessible for academic exploration.

---

## 📜 License
This project uses publicly available CVE data from the MITRE CVE List.  
All analysis and visualization code is licensed under the **MIT License**.

---

> _“Turning cybersecurity data into actionable intelligence is not just analytics — it’s resilience.”_
