[genioCE_org_profile_README.md](https://github.com/user-attachments/files/22061221/genioCE_org_profile_README.md)
# genioCE · Open Data Platform + Factory Intelligence (OSS)

**One line:** We ship a modular, open-source data platform (Kafka/MQTT → Iceberg/Trino → dbt/GE → OpenLineage) and **FactoryPack** (OEE · Trace · PdM · Vision) — with **read-only** OT mirrors and clear SLOs. Lower license spend, remove lock-in, and stay aligned with Purdue/IEC-62443.

---

## Why (grounded)

- **Cut run-rate:** Replace high-fee data tooling with mature OSS where it makes sense; keep proprietary only where it still earns its keep.
- **Open formats:** Iceberg tables + Kafka/MQTT keep your data portable across engines and clouds.
- **Safe for OT:** Mirror from PLC/RTU via OPC UA/Modbus into MQTT/TSDB/Lakehouse — **no control writes.**
- **Ops reality:** Jobs-as-code (dbt/Airflow), defined SLOs/MTTR, and rollback criteria — no big-bang cutovers.

---

## What we ship

### Data Platform (pick & choose)
- Core lakehouse: **Iceberg + Trino + Nessie + MinIO**  
  https://github.com/genioCE/g-oss-core
- Change data capture: **Debezium + Kafka/Redpanda + Iceberg sink**  
  https://github.com/genioCE/g-oss-cdc
- Batch/ELT + DQ: **Airflow + dbt + Great Expectations**  
  https://github.com/genioCE/g-oss-batch
- Lineage: **OpenLineage (Marquez)**  
  https://github.com/genioCE/g-oss-governance
- Observability: **Prometheus + Grafana + Loki + Tempo + OTel**  
  https://github.com/genioCE/g-oss-observability
- Search/logs: **OpenSearch + Dashboards**  
  https://github.com/genioCE/g-oss-search
- BI: **Apache Superset**  
  https://github.com/genioCE/g-oss-bi
- Security: **Keycloak (SSO) + Vault (secrets) + OPA (policy)**  
  https://github.com/genioCE/g-oss-security
- MFT: **SFTPGo + NiFi**  
  https://github.com/genioCE/g-oss-mft
- Streaming compute: **Apache Flink**  
  https://github.com/genioCE/g-oss-stream

### IT/OT add-ons (production-friendly)
- EDR/SIEM (Wazuh) — https://github.com/genioCE/g-oss-edr-siem  
- osquery Fleet (FleetDM) — https://github.com/genioCE/g-oss-fleet-osquery  
- NIDS (Zeek/Suricata) — https://github.com/genioCE/g-oss-nids  
- Backup/DR (restic/pgBackRest/Velero patterns) — https://github.com/genioCE/g-oss-backup-dr  
- Dev platform (Gitea/Drone/Harbor) — https://github.com/genioCE/g-oss-devops  
- ITSM/Assets/KB (GLPI/Snipe-IT/BookStack) — https://github.com/genioCE/g-oss-itsm  
- Comms (Mattermost + Matrix/Element) — https://github.com/genioCE/g-oss-comms  
- **OT mirror (read-only)** OPC UA/Modbus → MQTT/Kafka → TSDB/Lake — https://github.com/genioCE/g-oss-ot-mirror  
- IoT broker (EMQX) — https://github.com/genioCE/g-oss-iot-broker  
- Edge dataflow (Node-RED/NiFi/Telegraf) — https://github.com/genioCE/g-oss-edge-flow  
- TSDB analytics (VictoriaMetrics + QuestDB + Grafana) — https://github.com/genioCE/g-oss-tsdb  
- GIS (PostGIS + GeoServer) — https://github.com/genioCE/g-oss-gis

### FactoryPack (OEE · Trace · PdM · Vision)
- Meta-repo overlays: dbt packs, dashboards, pilot stack (EMQX · VictoriaMetrics · Grafana · Node-RED · Telegraf)  
  https://github.com/genioCE/g-oss-factorypack
- **Mirror-first** by design: Purdue-aligned, IEC-62443 friendly (no control writes).

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

## Try it in 5 minutes

- **Data platform (local POC):**
  1) https://github.com/genioCE/g-oss-platform  
  2) `make bootstrap && make up-poc` (Core + CDC + Batch + Governance)

- **IT/OT bundle (local pilots):**
  1) https://github.com/genioCE/g-oss-it-ot-platform  
  2) `make bootstrap && make up-it-core` or `make up-ot-core`

- **FactoryPack pilot (single line):**
  1) https://github.com/genioCE/g-oss-factorypack  
  2) `docker compose -f compose/docker-compose.pilot.yml up -d`  
     Optional: run `scripts/sim_mqtt_line.py` to simulate line data.

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
