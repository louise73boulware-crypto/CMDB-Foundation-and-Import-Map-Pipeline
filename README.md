# CMDB-Foundation-and-Import-Map-Pipeline
Automated system data pipeline that cleans and maps data cleanly.

## CMDB Data Pipeline & Ingestion Guardrails

An enterprise-grade ServiceNow integration pipeline built to automate data normalization, optimize configuration management database (CMDB) health, and prevent data corruption during bulk imports.

---

## 📋 Project Overview
Managing configuration item (CI) data at scale often means dealing with fragmented, poorly formatted data from third-party monitoring tools or legacy inventory spreadsheets. This project establishes an automated ingestion pipeline that transforms chaotic raw source data into standardized, audit-ready CMDB records.

This solution processes a dataset of **500 custom server records** featuring complex formatting issues, mixed casing, trailing whitespaces, and missing unique identifiers. It utilizes **Now Assist for Creator** to dynamically generate a low-level JavaScript normalization layer, coupled with strict platform architecture configurations to enforce database hygiene.

### 🧠 Core Competencies Demonstrated
* **System Import Sets & Staging Architecture** (Decoupling raw source data from core system tables)
* **Transform Mapping Logic** (Multi-tier target matrix routing)
* **Advanced Coalesce Strategy** (Handling identity collision and multi-field matching logic)
* **Generative AI Platform Development** (Leveraging Now Assist for automated script orchestration)
* **Data Integration Governance** (Preventing system record overwrites on empty keys)

---

## 🛠️ Architecture & Technical Design


### 1. Staging Data Tier
* **Source Table:** `Raw Server Import [u_raw_server_import]`
* **Target Destination:** `Server [cmdb_ci_server]`
* **Scope Isolation:** Built entirely inside the isolated `Server Ingestion Engine` custom application scope to adhere to strict architecture governance and enable clean native source control deployment.

### 2. Field Mapping & Normalization Engine
Data normalization handles high-velocity variance using an AI-generated JavaScript transformation script mapped to the `Name` attribute:

```javascript
answer = (function transformRow(source) {
    // Standardizes chaotic case variations and strips structural trailing white spaces
    var rawName = source.u_name.toString();
    return rawName.trim().toLowerCase(); 
})(source);
```

---

## 🛡️ Critical Design Choices

### 1. Handling Missing Unique Keys
**Setting Configured:** `Create new record on empty coalesce fields = TRUE`

**The Engineering Logic:** 
In typical configurations, leaving this setting `FALSE` causes the system to search the existing CMDB target table for any entry that *also* has an empty/blank coalesce field. If found, the data import engine treats it as a successful match and continuously overwrites that single empty record. This leads to severe data corruption. 

By setting this value to `TRUE`, the pipeline detects rows missing a unique serial identifier (e.g., `Server-Linux-03`), skips matching, and safely processes the row as a standalone new record awaiting manual administrative staging remediation.

### 2. Collision Remediation (Composite Coalescing)
During initial multi-record data validation testing over the 500-record batch, data collisions occurred where identical structural serial keys (e.g., `SN-61047`) represented distinctly different computing platforms across teams. 

**The Resolution:** 
The pipeline was scaled from a simple single-field match to a **Composite Coalesce Key**. By setting both `Name` AND `Serial Number` map matrices to `Coalesce = True`, a record is only updated if *both* credentials match perfectly. This preserves environment records from being inadvertently overwritten by cross-platform discovery tooling imports.

---

## 🚀 Execution & Verification Walkthrough

1. **Load Data Phase:** Ingested `Mock_Server_Data_500.csv` containing mixed casings (`SERVER-WINDOWS-002`, `SeRvEr-uBuNtU-003`) and extreme spacing padding.
2. **Execution Summary:** 
   * **Staging Status:** 500 rows processed.
   * **Transformation Profile:** Clean inserts, calculated updates, zero pipeline crashes.
3. **Database Inspection:** Running a filter query on `cmdb_ci_server.list` confirms that strings have successfully formatted into clean lower-case entries, trailing spacing is removed, and composite records match production infrastructure flawlessly.

| Source Data Sample | Target CMDB Result |
| :--- | :--- |
| `   server-solaris-002   ` | `server-solaris-002` |
| `SeRvEr-uBuNtU-003` | `server-ubuntu-003` |

---

## 📂 Repository Contents
* `/sys_app_...xml` - Scoped application descriptor and cross-scope table access profiles.
* `/sys_transform_map_...xml` - Configuration schema for the transformation pipeline mappings.
* `/sys_transform_script_...xml` - Now Assist Javascript source records.
* `/data_source/` - Mock sample datasets utilized to execute structural pipeline pressure-testing.
