# ⚙️ Bosch *Preventive Maintenance Dashboard

A Django-based **Preventive Maintenance Monitoring System** for CNC machines on the Bosch TEF32 shop floor. It visualises spindle temperature, current, following error, and part count in real time — and automatically sends email alerts when thresholds are breached.

---

## 📸 Features

- 📊 **Live graphs** — Temperature, Current, Following Error, Part Count per machine
- 🔢 **Multi-spindle support** — 1 to 4 spindles per machine, dynamically configured
- 🔴 **Alert detection** — Warning & Critical thresholds with badge indicators
- 🔔 **Alerts tab** — Live badge count, filter by type/severity, machine search
- 📧 **Batch email alerts** — Grouped emails every 15 minutes (no flooding)
- 🏭 **Machine detail page** — All metrics for one machine on a single scrollable view
- 🕐 **Shift filter** — Shift 1 / Shift 2 / Shift 3 / ALL + date picker
- ⚡ **Response caching** — Django LocMemCache to reduce DB load
- 👁️ **Show/Hide machines** — Per-tab machine visibility toggle

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend | Django 5.2 |
| Database | Microsoft SQL Server (`mssql-django`) |
| Charts | Plotly.js 2.26 |
| CSS | Bootstrap 5.3.2 |
| Icons | Bootstrap Icons 1.11 |
| Email | Django `EmailMultiAlternatives` via Bosch SMTP |
| Cache | Django LocMemCache |

---

## 📁 Project Structure

```
pmProject2026/
├── pmProject2026/
│   ├── settings.py          # DB, email, cache, static config
│   ├── urls.py              # Root URL config
│   └── config/
│       └── machine_config.py  # ⭐ Central machine configuration
├── accounts/
│   ├── views.py             # All views + API endpoints
│   ├── urls.py              # App-level routes
│   └── email_alerts.py      # Alert batching + HTML email builder
├── templates/
│   ├── machine_data.html    # Main dashboard (tabs, graphs, alerts)
│   ├── machine_detail.html  # Single-machine detail page
│   └── machineselection.html # Landing page
└── static/
    ├── css/
    │   ├── style1.css       # Styles for machine_data.html
    │   └── style2.css       # Styles for machine_detail.html
    └── images/
```

---

## ⚙️ Machine Configuration

All machine definitions live in `pmProject2026/config/machine_config.py` in the `MACHINE_CONFIG` dict.

### Structure

```python
MACHINE_CONFIG = {
    "chiron": {
        "tabs": ["ALL Machines", "temperature", "current", "partcount", "followingerror"],
        "machines_by_tab": {
            "temperature": ["ChironDZ1", "ChironDZ2"],
            # ...
        },
        "machines": {
            "ChironDZ1": {
                "labels": {"sp1": "SP1", "sp2": "SP2"},
                "temperature": {"sp1temp": "ColumnName1", "sp2temp": "ColumnName2"},
                "current":     {"sp1current": "ColumnName3", "sp2current": "ColumnName4"},
                "partcount":   ["PartCountColumn"],
                "followingerror": {
                    "axes":      {"a": "AAxisCol", "u": "UAxisCol"},
                    "setpoints": {"a": "ASetpointCol", "u": "USetpointCol"},
                }
            }
        }
    }
}
```

### Adding a New Machine

1. Open `machine_config.py`
2. Find the machine group (or create a new one)
3. Add the machine name to `machines_by_tab` for each relevant tab
4. Add its entry under `machines` with correct DB column names
5. If it's a new group, add it to the dropdown in `machineselection.html`

---

## 🚨 Alert Thresholds

| Alert Type | Warning | Critical |
|---|---|---|
| Temperature | ≥ 60 °C | ≥ 70 °C |
| Current | > 5 A | > 6 A |
| Following Error | > 0.015 mm | > 0.020 mm |
| Part Count | — | Any hour with **0 parts** |

> Thresholds are defined as JS constants at the top of `machine_data.html` and `machine_detail.html`.

---

## 📧 Email Alert System

Alerts are batched and sent in two groups:

| Group | Includes |
|---|---|
| `group_a` | Temperature + Current |
| `group_b` | Following Error + Part Count |

- **15-minute server-side cooldown** per group prevents alert flooding
- Recipients are configured in `accounts/email_alerts.py` → `ALERT_RECIPIENTS`
- SMTP relay: `rb-smtp-auth.rbesz01.com:25`

---

## 🔗 API Endpoints

| URL | Method | Purpose |
|---|---|---|
| `/` | GET | Machine group selection |
| `/machine-data/` | GET | Main dashboard page |
| `/machine-detail/` | GET | Single machine detail |
| `/get-shift-data/` | GET | JSON data API (graphs) |
| `/send-alert-email/` | POST | Collect alert into batch |
| `/flush-alert-email/` | POST | Send batched emails |

---

## 🚀 Setup & Installation

### Prerequisites

- Python 3.x
- ODBC Driver 17 for SQL Server (installed on OS)
- Network access to the Bosch MSSQL server and SMTP relay

### Install

```bash
# 1. Clone the repo
git clone <repo-url>
cd pmProject2026

# 2. Install dependencies
pip install django mssql-django

# 3. Verify settings.py — DB credentials and EMAIL settings

# 4. Run the development server
python manage.py runserver 0.0.0.0:8000
```

Open `http://<server-ip>:8000/` in your browser.


## ⚠️ Known Limitations

- **Alert batch state** (`_batch`, `_last_sent`) is in-memory — lost on server restart. For production, persist to DB or cache.
- **15-min flush is browser-driven** — closing the tab stops the flush timer.
- **`csrf_exempt`** is used on alert endpoints — acceptable for internal use only.


---

## 📋 Database

| Setting | Value |
|---|---|
| Server | `na0vm00024.apac.bosch.com` |
| Port | `1433` |
| Database | `DB_MFC2DB_SQL` |
| Driver | ODBC Driver 17 for SQL Server |

---

## 👤 Contact

| | |
|---|---|
| Project | Preventive Maintainence Dashboard — Bosch TEF32 (104,106) |
| Contact | Rupesh Suryawanshi |
| Version | 1.0 — April 2026 |
