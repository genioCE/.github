
# HNAI · Open Data Platform + Vertical Packs (Oil & Gas • Manufacturing)

**One line:** We ship a modular, open-source data platform (Kafka/MQTT → Iceberg/Trino → dbt/GE → OpenLineage) and vertical packs for **Oil & Gas (EnergyPack)** and **Manufacturing (FactoryPack)** — built on **read-only** OT mirrors with clear SLOs. Cut license spend, remove lock-in, and stay aligned with Purdue/IEC‑62443.

---

## Why (grounded)

- **Cut run-rate:** Replace high-fee tools with mature OSS where it’s ready; keep proprietary only where it still earns its keep.  
- **Open formats:** Iceberg tables + Kafka/MQTT keep your data portable across engines/clouds.  
- **Safe for OT:** Mirror from PLC/RTU via OPC UA/Modbus into MQTT/TSDB/Lakehouse — **no control writes.**  
- **Ops reality:** Jobs-as-code (dbt/Airflow), defined SLOs/MTTR, rollback criteria — no big-bang cutovers.

---

## Vertical packs (ready-to-deploy overlays)

### Oil & Gas — **EnergyPack** (Well Intelligence)
Outcomes: **well/field dashboards**, semantic wellfile search, **SCADA mirrors** for reliability, and fast ELT to lakehouse.

- **What’s inside (assembled from our OSS modules):**
  - **OT mirror (read-only):** OPC UA/Modbus → MQTT/Kafka → TSDB/Lake  
    https://github.com/genioCE/g-oss-ot-mirror
  - **TSDB analytics:** VictoriaMetrics + QuestDB + Grafana  
    https://github.com/genioCE/g-oss-tsdb
  - **Lakehouse & ELT:** Iceberg + Trino + dbt + Great Expectations  
    https://github.com/genioCE/g-oss-core • https://github.com/genioCE/g-oss-batch
  - **Lineage & Observability:** OpenLineage • Prom/Grafana/Loki/Tempo  
    https://github.com/genioCE/g-oss-governance • https://github.com/genioCE/g-oss-observability
  - **Search/Docs:** OpenSearch (+ wellfile indexing)  
    https://github.com/genioCE/g-oss-search
  - **Security:** Keycloak (SSO) + Vault (secrets) + OPA (policy)  
    https://github.com/genioCE/g-oss-security

- **Typical use cases & targets**
  - **Production dashboards** (pressures/rates) with P95 freshness ≤ **10 s**.  
  - **Semantic wellfile search** across PDFs/TIFs/CSV in Iceberg.  
  - **Change data** from field systems and accounting (CDC) for daily closes.

- **Try it fast**
  - Data platform POC: https://github.com/genioCE/g-oss-platform → `make bootstrap && make up-poc`  
  - OT/TSDB pilots: https://github.com/genioCE/g-oss-it-ot-platform → `make up-ot-core`

---

### Manufacturing — **FactoryPack** (OEE • Trace • PdM • Vision)
Outcomes: **OEE by shift**, downtime Pareto, **traceability** across stations, PdM alerts, and Vision QC aggregation — all mirror‑first.

- **Repo:** https://github.com/genioCE/g-oss-factorypack  
  Packs: dbt models (OEE/Trace/PdM/Vision), Grafana dashboards, single‑line pilot stack (EMQX · VictoriaMetrics · Grafana · Node‑RED · Telegraf).

- **Try it fast**
  - Pilot (single line): `docker compose -f compose/docker-compose.pilot.yml up -d`  
    Optional: `scripts/sim_mqtt_line.py` to simulate RUN/IDLE/STOP and counts.

- **Targets**
  - **OEE by shift** ≤ **5 min** after close; top downtime reasons; accuracy ±2% vs manual logs.  
  - **TSDB freshness** ≤ **10 s** (OPC UA → MQTT → TSDB).  
  - **Trace coverage** ≥ **95%** (ramping to 98% with scanners aligned).

---

## Core platform (pick & choose modules)

- Core lakehouse: **Iceberg + Trino + Nessie + MinIO** — https://github.com/genioCE/g-oss-core  
- CDC: **Debezium + Kafka/Redpanda + Iceberg sink** — https://github.com/genioCE/g-oss-cdc  
- Batch/ELT + DQ: **Airflow + dbt + Great Expectations** — https://github.com/genioCE/g-oss-batch  
- Lineage: **OpenLineage (Marquez)** — https://github.com/genioCE/g-oss-governance  
- Observability: **Prometheus + Grafana + Loki + Tempo + OTel** — https://github.com/genioCE/g-oss-observability  
- Search/logs: **OpenSearch + Dashboards** — https://github.com/genioCE/g-oss-search  
- BI: **Apache Superset** — https://github.com/genioCE/g-oss-bi  
- Security: **Keycloak (SSO) + Vault (secrets) + OPA (policy)** — https://github.com/genioCE/g-oss-security  
- MFT: **SFTPGo + NiFi** — https://github.com/genioCE/g-oss-mft  
- Streaming compute: **Apache Flink** — https://github.com/genioCE/g-oss-stream  

**IT/OT add‑ons (production‑friendly):**  
EDR/SIEM — https://github.com/genioCE/g-oss-edr-siem • FleetDM — https://github.com/genioCE/g-oss-fleet-osquery • NIDS — https://github.com/genioCE/g-oss-nids • Backup/DR — https://github.com/genioCE/g-oss-backup-dr • Dev platform — https://github.com/genioCE/g-oss-devops • ITSM/Assets/KB — https://github.com/genioCE/g-oss-itsm • Comms — https://github.com/genioCE/g-oss-comms • **OT mirror** — https://github.com/genioCE/g-oss-ot-mirror • IoT broker — https://github.com/genioCE/g-oss-iot-broker • Edge dataflow — https://github.com/genioCE/g-oss-edge-flow • TSDB — https://github.com/genioCE/g-oss-tsdb • GIS — https://github.com/genioCE/g-oss-gis

---

## Architecture (at a glance)

```
[ DBs ]  [ Files/SFTP ]  [ OT Mirrors (OPC UA/Modbus) ]
   |          |                     |
   |      NiFi/SFTPGo           Telegraf/Node-RED  →  MQTT/Kafka  →  TSDB (hot)
   |                       \                            |
   +-- Debezium (CDC) ------+---------------------------+------→  Iceberg Tables on MinIO (Nessie)
                                      |
                                   Trino SQL  ← dbt (ELT)  → Great Expectations (DQ gates)
                                               \→ OpenLineage (Marquez) lineage
                                   Superset / APIs / Notebooks / Grafana

Observability: Prometheus + Grafana + Loki + Tempo      Security: Keycloak (SSO) + Vault (secrets) + OPA (policy)
```

---

## SLOs & KPIs (defaults you can adopt)

| Metric                     | Phase 1        | Phase 2        | Notes                                |
|---                         |---:            |---:            |---                                   |
| Batch freshness            | ≤ 2h           | ≤ 30m          | Source close → availability          |
| Stream latency (P95)       | ≤ 60s          | 10–30s         | CDC/stream end-to-end                |
| Pipeline success rate      | ≥ 99.5%        | ≥ 99.9%        | Excluding planned maintenance        |
| MTTR (failed run)          | < 30m          | < 15m          | Runbooks + on-call                   |
| Data completeness          | ≥ 99.7%        | ≥ 99.9%        | Counts/continuity                    |
| Data accuracy              | ≥ 99.5%        | ≥ 99.8%        | Ranges/ref integrity                 |
| OT TSDB freshness          | ≤ 10s          | ≤ 3s           | OPC-UA/MQTT → TSDB path              |
| Security coverage          | SSO/Vault      | + OPA          | All UIs behind SSO; secrets rotated  |

---

## Safety & compliance

- **Read-only OT mirrors**; no write paths to PLC/HMI.  
- Purdue-aligned segmentation; OT DMZ boundary.  
- **SSO** (Keycloak) and **secrets** (Vault) everywhere; **policy-as-code** (OPA).  
- Audit & lineage: OpenLineage + Nessie commits + dbt tests.

---

## Procurement

We support standard Git/Helm flows. Azure **Kubernetes app (AKS extension)** CNAB packaging is in progress for selected modules/FactoryPack.

---

## Support & licensing

- License: Apache-2.0 across modules unless stated otherwise.  
- Support:
  - Community: issues & discussions in each repo.  
  - Commercial: Bronze (business-hours), Silver (24×5), Gold (24×7).

**Contact:** brian@hewesguyen.com
