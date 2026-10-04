# SafeLine WAF → Wazuh SIEM Security Monitoring Lab

A hands-on web security and SIEM integration lab built in a controlled home environment using SafeLine WAF, DVWA, Apache, Kali Linux, PostgreSQL, Python, and Wazuh.

The project evolved in two phases:

1. **Phase 1 — SafeLine WAF + DVWA**
2. **Phase 2 — SafeLine WAF → Wazuh SIEM integration**

## Objectives

- Deploy an intentionally vulnerable web application for controlled testing
- Understand WAF reverse-proxy architecture
- Generate security-testing traffic from Kali Linux
- Validate SafeLine WAF detections
- Extract SafeLine detection events from PostgreSQL
- Convert WAF events into structured JSON
- Ingest the events into Wazuh
- Create a custom Wazuh detection rule
- Prevent duplicate event export using state tracking
- Automate event collection with systemd
- Validate the complete attack-to-detection pipeline

## Final Architecture

```text
Kali Linux
    │
    │ Controlled security-testing traffic
    ▼
SafeLine WAF
    │
    ▼
DVWA / Apache
    │
    ▼
SafeLine PostgreSQL
    │
    ▼
Python Log Exporter
/usr/local/bin/safeline-to-wazuh.py
    │
    ▼
/var/log/safeline/events.json
    │
    ▼
Wazuh JSON Decoder
    │
    ▼
Custom Rule 100100
Level 10
    │
    ▼
Wazuh Dashboard / Alert
```

## Technologies

- Ubuntu
- Kali Linux
- SafeLine WAF
- Docker
- Apache
- DVWA
- PostgreSQL
- Python
- JSON
- Wazuh SIEM
- systemd

## Project Phases

### Phase 1 — SafeLine WAF + DVWA

Built a web security lab in which SafeLine operates as a reverse proxy/WAF in front of DVWA hosted by Apache.

[Phase 1 documentation](phase-1-safeline-waf/documentation/setup-and-testing.md)

### Phase 2 — SafeLine WAF → Wazuh

Extended the WAF lab by reading new SafeLine detection events from PostgreSQL, converting them to JSON, writing them to `/var/log/safeline/events.json`, and allowing Wazuh to detect them with custom Rule 100100.

[Phase 2 documentation](phase-2-wazuh-integration/documentation/integration.md)

## Detection Pipeline

```text
Security-testing traffic
        ↓
SafeLine WAF
        ↓
PostgreSQL detection event
        ↓
Python exporter
        ↓
Structured JSON
        ↓
Wazuh JSON decoder
        ↓
Rule 100100 / Level 10
        ↓
Alert
```

## Validation

The custom rule was validated using `wazuh-logtest`.

The test returned:

```text
id: '100100'
level: '10'
description: 'SafeLine WAF detected a web security event'
Alert to be generated.
```

## Repository Structure

```text
SafeLine-WAF-DVWA-Security-Lab/
├── README.md
├── .gitignore
├── .env.example
├── phase-1-safeline-waf/
│   ├── architecture/
│   │   └── architecture.md
│   └── documentation/
│       └── setup-and-testing.md
└── phase-2-wazuh-integration/
    ├── architecture/
    │   └── integration-architecture.md
    ├── exporter/
    │   └── README.md
    ├── wazuh-rules/
    │   └── custom-rule-example.xml
    └── documentation/
        └── integration.md
```

Screenshots and architecture images can be added later.

## Disclaimer

This project was created for educational purposes in a controlled home-lab environment. Security testing was performed only against systems under my control or intentionally vulnerable applications such as DVWA.

No real organization infrastructure or production systems were targeted.
