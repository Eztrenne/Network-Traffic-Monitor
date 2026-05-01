<div align="center">

# 🛡️ Network Traffic Monitoring Dashboard
### Cyber Attack Detection & Analysis · Power BI

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-00FF87?style=for-the-badge)

*A network traffic monitoring dashboard designed to detect ICMP Flood and TCP Flood attacks, built as a final project for a Data Analyst course.*

---

**[🎭 How It Was Built](#-fictional-scenario--how-it-was-built) · [🖥️ Dashboard](#️-dashboard-overview) · [📁 Data Model](#-data-model) · [📐 DAX Measures](#-dax-measures) · [🌍 Real World](#-real-world-application) · [📬 Contact](#-contact)**

</div>

---

## 📌 Project Overview

This project simulates a **network security monitoring system** built entirely in Power BI. Starting from a real Wireshark packet capture, the dataset was enriched with simulated attack scenarios to create a analytically rich environment where ICMP Flood and TCP Flood attacks can be visually detected through interactive KPIs and dashboards.

The system monitors over **4,750 network connections** across **4 years (2022–2025)**, identifying:
- 🔴 **ICMP Flood attacks** — threshold: ≥ 5 packets/day
- 🟡 **TCP Flood attacks** — threshold: ≥ 8 packets/day
- ⚠️ **Malicious IPs** — both source and destination
- 🌍 **Geographic origin** of attacks — 9 countries tracked

---

## 🎭 Fictional Scenario — How It Was Built

> This project was developed in a **fictional but realistic scenario** to simulate how a Data Analyst would approach network security monitoring in a real organization.

---

> 💡 *The diagram below illustrates the full build process visually:*

<img width="1317" height="607" alt="image" src="https://github.com/user-attachments/assets/ce34e1a1-e698-48fe-b274-7a5f8eb604b8" />


---

### Step 1 — Raw Data Collection

A network packet capture was performed using **Wireshark** and exported as a `.csv` file. The raw dataset contained real network traffic with the following fields:

| Column | Description |
|---|---|
| `No.` | Packet number |
| `Time` | Timestamp of the packet |
| `Source` | Origin IP address |
| `Destination` | Destination IP address |
| `Protocol` | Network protocol used (TCP, ICMP, DNS...) |
| `Length` | Size of the packet in bytes |
| `Info` | Wireshark packet description |

> 💡 The raw dataset had **393,511 rows** — a realistic volume of real network traffic.

---

### Step 2 — Data Enrichment & Simulation

Since the raw dataset lacked temporal depth, geographic context and attack examples, **Python (pandas)** was used to process and enrich the data:

- **Reduced** the dataset to **4,750 representative rows**, prioritizing rows where `Source` was a valid IP address and removing malformed entries
- **Dropped** the `Time` and `Info` columns — `Time` was replaced by a richer `Date` column and `Info` had no analytical value
- **Added a `Date` column** with dates randomly but equitably distributed across 2022–2025, simulating 4 years of network monitoring
- **Simulated 75 ICMP Flood attack rows** concentrated on specific dates across all 4 years
- **Simulated 75 TCP Flood attack rows** on different specific dates, some spanning 2 consecutive days to mimic prolonged attacks
- **Injected 10 suspicious IPs** (5 source attackers, 5 destination targets) distributed across the simulated attack rows
- **Added a `Country` column** with geographically coherent mapping — same IP always gets the same country, with Spain as the dominant country since the monitoring is Spain-based
- **Fixed malformed IPs** — the original Wireshark export had some IPs with missing dots (e.g. `1921677162` instead of `192.167.7.162`) which were corrected programmatically

> 💡 **Interesting detail:** The country distribution was intentionally weighted — Spain ~19%, US ~18%, China ~16%, Russia ~13% — reflecting a realistic geopolitical threat landscape where attacks come primarily from Russia and China toward a Spanish network.

---

### Step 3 — Star Schema Modeling in Power BI

The enriched `.csv` was imported into **Power BI** and split into multiple tables using **Power Query (M language)**, building a proper **Star Schema** without ever leaving Power BI:

- Each dimension table was created by **duplicating the original table** and removing all columns except the relevant one
- **Duplicates were removed** from each dimension to ensure uniqueness
- **Index columns were added** as surrogate keys (ID_IP_Source, ID_Country, etc.)
- **Conditional columns** were added using M formulas — for example, `TYPE_RISK_S` and `TYPE_RISK_D` were created using `List.Contains()` to flag the 10 known malicious IPs with value `1`
- The **Calendar table** was created directly in DAX using `ADDCOLUMNS` + `CALENDAR()` — no external file needed
- **Relationships** were established in Model View connecting all dimension tables to the central fact table

> 💡 **Interesting detail:** Power Query's `Table.NestedJoin` (Merge Queries) was used to bring the surrogate keys back into the fact table, replacing the original text columns (Source, Destination, Protocol, Country) with their corresponding integer IDs — making the model leaner and faster.

---

### Step 4 — Dashboard Development

Three analytical views were built in Power BI, all sharing the same **Matrix-style dark theme** (black background, green text) to reinforce the cybersecurity aesthetic:

- **View 1** — Protocol-level monitoring (ICMP & TCP traffic over time and by country)
- **View 2** — IP-level monitoring with malicious IP detection
- **View 3** — Time intelligence analysis (YoY, MoM, LY) + geographic map

---

### Step 5 — Attack Detection Logic

This is where the analytical logic lives. The detection system works on two levels:

**Level 1 — Threshold-based KPI alerts**
DAX measures detect when daily packet counts exceed the attack thresholds and change the KPI color using **conditional formatting rules**:

- `Color_ICMP` → returns `1` (red) if a single date is selected AND Total ICMP ≥ 5
- `Color_TCP` → returns `1` (red) if a single date is selected AND Total TCP ≥ 8
- `Color_IP_S` / `Color_IP_D` → returns `1` (red) if malicious IPs were active on the selected date

> 💡 **Key design decision:** `HASONEVALUE()` was used in all color measures so that KPIs stay **green by default** when no specific date is selected — avoiding false alarms on the global view.

**Level 2 — Geographic attack origin**
When a user clicks on an attack spike in the time chart, the country bar chart updates through **cross-filtering**, revealing which country the attack originated from. In most cases Russia and China dominate, consistent with the malicious IP assignments.

---

## 🖥️ Dashboard Overview

### View 1 — ICMP & TCP Traffic

| Element | Description |
|---|---|
| 📊 KPI — Total ICMP | Turns red when ≥ 5 ICMP packets on selected date |
| 📊 KPI — Total TCP | Turns red when ≥ 8 TCP packets on selected date |
| 📅 Date card | Shows selected date (`HASONEVALUE` + `FORMAT`) |
| 📈 Line chart (left) | ICMP & TCP over time — attack spikes visible |
| 📊 Bar chart (right) | ICMP & TCP by country — identifies attack origin |
| 🔽 Protocol slicer | Filter by TCP, ICMP or all |
| 🌍 Country slicer | Filter by country of origin |

### View 2 — IP Monitoring

| Element | Description |
|---|---|
| 📊 KPI — Total IP Source | Turns red if malicious source IPs active on date |
| 📊 KPI — Total IP Destination | Turns red if malicious destination IPs active on date |
| 📈 Bar chart (top) | IP activity by country |
| 📈 Bar chart (bottom) | IP activity over time |
| 🔽 Risk slicer | Toggle all traffic / attack-only (TYPE_RISK = 1) |

### View 3 — Technical Analysis & Map

| Element | Description |
|---|---|
| 📊 YoY, MoM, LY metrics | Compare attack volume across time periods |
| 🗺️ Map | Geographic distribution of connections |
| 📋 Table | Row-level data with conditional row formatting |
| 🔁 Bookmark toggle | Switch between table and map without leaving the view |
| 🔽 Metric slicer | Parameter-based selector to switch between YoY / MoM / LY |

---

## 🚨 Simulated Attack Dates

Use these dates in the date slicer to trigger red alerts:

### ICMP Flood
| Year | Dates |
|---|---|
| 2022 | March 14–15, September 7 |
| 2023 | January 22, June 3 |
| 2024 | April 10–11, November 20 |
| 2025 | February 5, August 17 |

### TCP Flood
| Year | Dates |
|---|---|
| 2022 | May 20–21, October 30 |
| 2023 | March 8, July 15–16 |
| 2024 | February 19, August 5 |
| 2025 | April 11, September 25–26 |

### Malicious IPs
| Type | IPs | Country |
|---|---|---|
| Source | `45.33.32.156`, `185.220.101.45`, `194.165.16.76`, `91.108.4.200`, `103.21.244.0` | Russia / China / US |
| Destination | `10.0.0.99`, `172.16.100.5`, `192.168.99.254`, `10.10.10.10`, `172.31.255.254` | Spain |

---

## 📁 Data Model

### Star Schema

<img width="1267" height="682" alt="image" src="https://github.com/user-attachments/assets/c40b0cbd-e11f-4e4b-a005-75512730e6d6" />

### Tables

| Table | Rows | Description |
|---|---|---|
| `FACT_Trafic` | 4,750 | Central fact table — all FKs + Date + Length |
| `DIM_IP_Source` | 137 | Unique source IPs + TYPE_RISK_S flag |
| `DIM_IP_Destination` | 245 | Unique destination IPs + TYPE_RISK_D flag |
| `DIM_Protocol` | 8 | Protocols + type (Normal / Flood / Cifrado) |
| `DIM_Country` | 9 | Countries + ID |
| `Dim_Calendario` | 1,461 | Full date table 2022–2025 via DAX |
| `Filtro_Ataque` | 2 | Helper table for attack/all slicer (no relationships needed) |

### Calendar Table (DAX)
```dax
Dim_Calendario = ADDCOLUMNS(
    CALENDAR(DATE(2022,1,1), DATE(2025,12,31)),
    "Año", YEAR([Date]),
    "Mes", MONTH([Date]),
    "NombreMes", FORMAT([Date], "MMMM"),
    "Trimestre", "Q" & QUARTER([Date]),
    "DiaSemana", FORMAT([Date], "DDDD"),
    "NumSemana", WEEKNUM([Date])
)
```

---

## 📐 DAX Measures

### Base Measures
```dax
Total_ICMP = CALCULATE(
    COUNTROWS(FACT_Trafic),
    FILTER(FACT_Trafic, RELATED(DIM_Protocol[Protocol]) = "ICMP")
)

Total_TCP = CALCULATE(
    COUNTROWS(FACT_Trafic),
    FILTER(FACT_Trafic, RELATED(DIM_Protocol[Protocol]) = "TCP")
)

Total_Conexiones = COUNTROWS(FACT_Trafic)

Total_IP_S = DISTINCTCOUNT(FACT_Trafic[ID_IP_Source_F])
Total_IP_D = DISTINCTCOUNT(FACT_Trafic[DIM_IP_Destination_F])
```

### Conditional Color Logic (KPI Alerts)
```dax
-- Returns 1 (red) only when a single date is selected AND threshold exceeded
-- Stays 0 (green) on global view to avoid false alarms

Color_ICMP = IF(HASONEVALUE(FACT_Trafic[Date]), IF([Total_ICMP] >= 5, 1, 0), 0)
Color_TCP  = IF(HASONEVALUE(FACT_Trafic[Date]), IF([Total_TCP]  >= 8, 1, 0), 0)
Color_IP_S = IF(HASONEVALUE(FACT_Trafic[Date]), IF([IPs_Maliciosas_S] >= 1, 1, 0), 0)
Color_IP_D = IF(HASONEVALUE(FACT_Trafic[Date]), IF([IPs_Maliciosas_D] >= 1, 1, 0), 0)
```

### Malicious IP Detection
```dax
IPs_Maliciosas_S = CALCULATE(
    COUNTROWS(FACT_Trafic),
    FILTER(DIM_IP_Source, DIM_IP_Source[TYPE_RISK_S] = "1")
)

IPs_Maliciosas_D = CALCULATE(
    COUNTROWS(FACT_Trafic),
    FILTER(DIM_IP_Destination, DIM_IP_Destination[TYPE_RISK_D] = "1")
)
```

### Time Intelligence
```dax
LY = CALCULATE(
    [Total_Conexiones],
    ALL(Dim_Calendario),
    Dim_Calendario[Año] = MAX(Dim_Calendario[Año]) - 1
)

MoM =
VAR T  = SELECTEDVALUE(Dim_Calendario[Mes])
VAR Ta = SELECTEDVALUE(Dim_Calendario[Año])
VAR _mes  = IF(T = 1, 12, T - 1)
VAR _anio = IF(T = 1, Ta - 1, Ta)
RETURN CALCULATE(
    [Total_Conexiones],
    ALL(Dim_Calendario),
    Dim_Calendario[Mes] = _mes,
    Dim_Calendario[Año] = _anio
)

YoY   = [Total_Conexiones] - [LY]
YoY % = DIVIDE([YoY], [LY], 0)
```

---

## 🌍 Real World Application

> This project was built in a fictional scenario, but the architecture and logic can be directly applied to a **real production environment** with minimal changes.

### Fictional vs Real — Component Mapping

| This Project | Real World Equivalent |
|---|---|
| Static CSV from Wireshark | Live feed from a **SIEM** (Splunk, Microsoft Sentinel) |
| Manually assigned dates | Real timestamps from network logs |
| Simulated attack rows | Actual anomalies detected by IDS/IPS systems |
| Hardcoded malicious IPs | **Threat intelligence feeds** (AbuseIPDB, VirusTotal API) |
| Python script for enrichment | **Automated ETL pipeline** (Azure Data Factory, Apache Airflow) |
| Power BI manual refresh | **DirectQuery** or **scheduled refresh** to live database |
| Static country mapping | **GeoIP lookup API** (MaxMind, ip-api.com) |
| Power Query star schema | Dedicated **data warehouse** (Azure Synapse, Snowflake) |

### Automated Real-Time Pipeline

```
Network Traffic
      │
      ▼
Packet Capture (Wireshark / NetFlow / Zeek)
      │
      ▼
Ingestion Layer (Python / Azure Data Factory / Kafka)
      │
      ▼
Storage (Azure SQL / PostgreSQL / Data Lake)
      │
      ▼
Transformation (dbt / Power Query / Python)
      │
      ▼
Power BI — DirectQuery / Scheduled Refresh
      │
      ▼
Live Alerts → Email / Microsoft Teams / PagerDuty
```

### Key Production Improvements

- **Real-time alerting** — Power BI integrated with Microsoft Teams or email to push notifications automatically when a KPI threshold is exceeded, no manual monitoring needed
- **Automated threat intelligence** — source IPs cross-referenced against live databases (AbuseIPDB, Shodan) to flag malicious IPs dynamically instead of hardcoding them
- **ML-based anomaly detection** — models like Isolation Forest or LSTM to catch unusual traffic patterns that don't fit simple threshold rules
- **Role-based access control** — Power BI workspace with different views for security analysts, managers and executives
- **Scalability** — swap the CSV for a proper database (Azure SQL, PostgreSQL) to handle millions of packets per day

---

---

## 📂 Repository Structure

```
📦 network-monitoring-dashboard
 ┣ 📂 data
 ┃ ┣ 📄 FACT_Trafic.csv
 ┃ ┣ 📄 DIM_IP_Source.csv
 ┃ ┣ 📄 DIM_IP_Destination.csv
 ┃ ┣ 📄 DIM_Protocol.csv
 ┃ ┣ 📄 DIM_Country.csv
 ┃ ┗ 📄 Midterm_53_group_modified.csv
 ┣ 📂 docs
 ┃ ┗ 📄 https://prezi.com/view/wTECHcFr9KKxQg6VeJuP/?referral_token=G7ItsYlnB3FN
 ┣ 📄 Network-Traffic-Monitor.pbix
 ┗ 📄 README.md
```

---

## 🛠️ Tech Stack

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

---

## 📬 Contact

<div align="center">

**Eztrenne** · Telecommunications Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/luis-f-choque-paca-0b6324240/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Eztrenne)

*Feel free to reach out for questions, feedback or collaboration.*

---

*Built with 💚 by Eztrenne · Telecommunications Engineer*

</div>
---
---
<div align="center">
    *Luis Fernando Choque Paca*
</div>
---
---
