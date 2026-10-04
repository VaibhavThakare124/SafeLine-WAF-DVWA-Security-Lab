# Phase 2 — SafeLine WAF → Wazuh SIEM Integration

## Objective

The objective of Phase 2 was to extend the SafeLine WAF lab by centralizing SafeLine security events inside Wazuh SIEM.

Phase 1 already provided:

```text
Kali → SafeLine → DVWA
```

SafeLine was successfully detecting and recording web security events.

The goal was to move these events into Wazuh for centralized monitoring and detection.

---

## 1. Python Exporter — The Bridge

SafeLine stores its detected events in its PostgreSQL database.

Wazuh does not directly read those SafeLine database records in this setup.

A custom Python exporter was therefore created:

```text
SafeLine PostgreSQL
       ↓
Python Exporter
       ↓
/var/log/safeline/events.json
       ↓
Wazuh
```

The exporter is installed at:

```text
/usr/local/bin/safeline-to-wazuh.py
```

The exporter:

- Connects to SafeLine's PostgreSQL database
- Finds new WAF detection events
- Converts each event into JSON
- Writes the JSON event to `/var/log/safeline/events.json`

---

## 2. State Tracking and Duplicate Prevention

The exporter uses:

```text
/var/lib/safeline-to-wazuh.state
```

to remember the last event ID it exported.

This prevents the same event from being repeatedly sent to Wazuh.

The simplified workflow is:

```text
Read last exported ID
        ↓
Query SafeLine for newer events
        ↓
Export new events
        ↓
Update state file
```

This makes the exporter incremental rather than repeatedly processing the complete event history.

---

## 3. JSON Output

The exporter converts SafeLine records into structured JSON.

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

The structure provides Wazuh with individual fields that can be decoded and used by detection rules.

Important fields include:

- `source`
- `src_ip`
- `host`
- `url_path`
- `attack_type`
- `risk_level`
- `action`
- `rule_id`
- `timestamp`

---

## 4. Wazuh JSON Processing

The generated event is written to:

```text
/var/log/safeline/events.json
```

Wazuh monitors this file and processes each JSON record.

The event fields can then be extracted individually by the JSON decoder.

Conceptually:

```text
JSON Event
    ↓
Wazuh JSON Decoder
    ↓
source = safeline
src_ip = ...
url_path = ...
attack_type = ...
risk_level = ...
rule_id = ...
```

---

## 5. Rule 100100 — The Detection Rule

The JSON event itself is log data.

A custom Wazuh detection rule was created to identify SafeLine events.

```text
Rule ID: 100100
Level: 10
Description: SafeLine WAF detected a web security event
```

The rule logic is:

> If the incoming JSON event has `source = safeline`, treat it as a SafeLine WAF security event.

The resulting flow is:

```text
SafeLine JSON event
       ↓
Wazuh JSON decoder
       ↓
Rule 100100
       ↓
Level 10 Alert
```

---

## 6. Rule Validation with wazuh-logtest

The rule was validated using `wazuh-logtest`.

The test returned:

```text
id: '100100'
level: '10'
description: 'SafeLine WAF detected a web security event'
Alert to be generated.
```

This confirmed that Wazuh successfully processed the SafeLine JSON event and matched the custom detection rule.

---

## 7. Automation

The Python exporter was automated using a Linux `systemd` service and timer.

The timer was configured to execute the exporter every 10 seconds.

This provided periodic collection of newly generated SafeLine events without manually running the Python script each time.

The automated flow was:

```text
SafeLine Event
      ↓
PostgreSQL
      ↓
Python Exporter
      ↓
events.json
      ↓
systemd Timer
      ↓
Wazuh
      ↓
Rule 100100
      ↓
Alert
```

---

## 8. End-to-End Testing

Controlled security-testing traffic was generated from Kali Linux against the lab DVWA application.

The complete pipeline was then validated:

```text
Kali Linux
    ↓
SafeLine WAF
    ↓
DVWA / Apache
    ↓
SafeLine PostgreSQL
    ↓
Python Exporter
    ↓
/var/log/safeline/events.json
    ↓
Wazuh JSON Decoder
    ↓
Rule 100100
    ↓
Level 10 Alert
    ↓
Wazuh Dashboard
```

The test successfully demonstrated that SafeLine detections could be extracted from PostgreSQL, transformed into JSON, ingested by Wazuh, decoded, and matched by a custom detection rule.

---

## 9. Key Learning Outcomes

This project provided practical experience with:

- Web Application Firewall monitoring
- WAF-to-SIEM integration
- PostgreSQL event extraction
- Python log processing
- Incremental event collection
- JSON security logging
- Wazuh Log Collector
- JSON decoding
- Custom Wazuh detection rules
- `wazuh-logtest`
- Linux systemd services and timers
- Centralized security monitoring
- End-to-end security event pipelines

---

## 10. Final Result

The original WAF lab was upgraded into a complete security monitoring pipeline:

```text
Kali
 ↓
SafeLine
 ↓
DVWA
 ↓
PostgreSQL
 ↓
Python Exporter
 ↓
JSON
 ↓
Wazuh
 ↓
Custom Rule 100100
 ↓
Level 10 Alert
 ↓
Dashboard
```

The project demonstrates how a WAF detection can be extracted, normalized, transported, decoded, and turned into a centralized SIEM alert.

---

## Disclaimer

This project was created in a controlled home-lab environment for educational and cybersecurity learning purposes. Testing was performed only against systems under my control or intentionally vulnerable applications such as DVWA.
