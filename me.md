## ✅ Monitoring & Observability Checklist for Stibo STEP

### 1. **Infrastructure Monitoring (Azure VMs, OS, DB, Network)**

* [ ] **CPU, memory, disk utilization** of app & DB servers.
* [ ] **Disk I/O, network I/O, latency** (especially between app servers and DB servers).
* [ ] **VM availability & uptime** (Azure Monitor / Splunk integration).
* [ ] **Azure resource metrics** (VM health, storage performance, network bandwidth).
* [ ] **DB monitoring** (Oracle/SQL Server/Postgres depending on STEP setup):

  * Query response times.
  * Slow queries / locks.
  * Connection pool usage.
  * Backup success/failure.
  * Transaction log growth.

---

### 2. **Application Server Monitoring (STEP App Servers)**

* [ ] **JVM monitoring** (heap usage, garbage collection times, thread counts, stuck threads).
* [ ] **STEP process monitoring**:

  * OIEP (Outbound Integration Endpoints) throughput & queue sizes.
  * Background job execution times (imports, exports, BGP, workflows).
  * Number of worker threads active vs idle.
  * Job failures / retries.
* [ ] **Service availability**:

  * STEP web UI.
  * STEP APIs.
  * Scheduler processes.

---

### 3. **Application-Level Observability**

* [ ] **Custom metrics / logs** pushed to Splunk:

  * Import/export success vs failure counts.
  * Time taken for imports/exports.
  * Workflow step completion times.
  * Failed object processing logs.
* [ ] **Business process KPIs**:

  * Number of new products created daily.
  * Failed validations.
  * Objects stuck in workflow.
* [ ] **User activity monitoring**:

  * Login success/failures.
  * Concurrent sessions.
  * API call volumes by integration.

---

### 4. **Integration Monitoring**

* [ ] **Inbound integrations**:

  * File arrival SLA monitoring (Azure Blob/FTP/SFTP).
  * Parsing & validation errors.
* [ ] **Outbound integrations**:

  * API call failures (HTTP codes, latency).
  * Queues filling up (MQ, Kafka, Azure Service Bus if used).
  * Downstream acknowledgements missing.
* [ ] **Third-party systems availability** (ERP, DAM, eCommerce).

---

### 5. **Logging & Alerting Strategy**

* [ ] **Standard log levels** (INFO, WARN, ERROR) consistently applied in STEP.
* [ ] **Centralized logging** (Splunk collecting from STEP app logs, DB logs, integration logs).
* [ ] **Correlation IDs** across transactions to trace a product lifecycle across systems.
* [ ] **Threshold-based alerts**:

  * High JVM heap usage.
  * Queue backlog beyond threshold.
  * Failed jobs > X in Y minutes.
* [ ] **Anomaly detection** (using Splunk ITSI or Azure Monitor insights).

---

### 6. **Security & Compliance Monitoring**

* [ ] Failed login attempts / brute force.
* [ ] Unauthorized API access attempts.
* [ ] Role/permission changes in STEP.
* [ ] Certificates / SSL expiry.

---

### 7. **Business & End-User Experience Monitoring**

* [ ] Synthetic transactions (log into STEP, run search, open workflow) to monitor end-user response times.
* [ ] API response times from consumer perspective.
* [ ] Dashboard for business KPIs (products published, workflows completed, SLA breaches).

---

### 8. **Review & Continuous Improvement**

* [ ] Regular review of Splunk dashboards and alerts with application teams.
* [ ] Post-incident reviews → update monitoring gaps.
* [ ] Capacity trend analysis (CPU, memory, jobs per day).
* [ ] Forecasting growth (DB size, product count, workflow volume).
