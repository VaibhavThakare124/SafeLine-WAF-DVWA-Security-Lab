# SafeLine → Wazuh Python Log Exporter

## Purpose

Wazuh does not directly read the SafeLine PostgreSQL detection records in this setup.

A custom Python exporter was developed as the bridge between SafeLine and Wazuh.

```text
SafeLine PostgreSQL
       ↓
Python Exporter
       ↓
/var/log/safeline/events.json
       ↓
Wazuh
```

## Script

The exporter is installed as:

```text
/usr/local/bin/safeline-to-wazuh.py
```

## What the Exporter Does

The Python script performs three main functions:

1. Connects to SafeLine's PostgreSQL database.
2. Finds new WAF detection events.
3. Converts each event into JSON and writes it to:

```text
/var/log/safeline/events.json
```

## Duplicate Prevention

The exporter maintains a state file:

```text
/var/lib/safeline-to-wazuh.state
```

The state file stores the last event ID exported by the script.

This allows the exporter to process only new events instead of repeatedly exporting the same records.

Conceptually:

```text
Last exported ID = 5

Database:
1  2  3  4  5  6  7
             ↑  ↑  ↑
             old new new

Exporter sends:
6, 7
```

## JSON Output

The exporter produces structured security events.

A sanitized example:

```json
{
  "source": "safeline",
  "id": 5,
  "event_id": "SANITIZED_EVENT_ID",
  "src_ip": "SANITIZED_IP",
  "host": "dvwa.local",
  "url_path": "/DVWA/vulnerabilities/csrf/",
  "attack_type": 2,
  "risk_level": 1,
  "action": 0,
  "rule_id": "m_csrf",
  "timestamp": 1791014222
}
```

## Important Fields

| Field | Meaning |
|---|---|
| `source` | Identifies the event source as SafeLine |
| `id` | SafeLine event/database identifier |
| `event_id` | SafeLine event identifier |
| `src_ip` | Source IP that generated the request |
| `host` | Target application host |
| `url_path` | Requested URL path |
| `attack_type` | SafeLine attack classification |
| `risk_level` | SafeLine risk classification |
| `action` | SafeLine action value |
| `rule_id` | SafeLine rule that detected the event |
| `timestamp` | Event timestamp |

## Interview Explanation

> I developed a Python-based log exporter that reads new SafeLine WAF events from PostgreSQL and converts them into JSON logs that Wazuh can ingest.

## Security

Do not publish:

- PostgreSQL passwords
- API keys
- tokens
- private credentials
- sensitive host information

Use environment variables or local configuration files excluded through `.gitignore`.
