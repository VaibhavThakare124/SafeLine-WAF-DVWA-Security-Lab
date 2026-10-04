# Phase 1 — SafeLine WAF Architecture

## Architecture

```text
Kali Linux
    │
    │ Controlled security-testing traffic
    ▼
SafeLine WAF
    │
    │ Reverse Proxy
    ▼
DVWA / Apache
    │
    ▼
Intentionally Vulnerable Web Application
```

## Components

### Kali Linux

Used to generate controlled security-testing traffic against the lab application.

### SafeLine WAF

Acts as the reverse proxy and inspects incoming web traffic before it reaches the application.

### DVWA

Damn Vulnerable Web Application, used as the intentionally vulnerable target for security testing.

### Apache

Web server hosting DVWA.

## Traffic Flow

1. Kali generates test traffic.
2. Traffic reaches SafeLine.
3. SafeLine inspects the request.
4. SafeLine applies its WAF detection logic.
5. Relevant traffic is handled according to the WAF decision.
6. SafeLine records security events.
7. The events become the input for the later SIEM integration.

## Security Boundary

All testing was performed inside a controlled home lab. DVWA was intentionally deployed as a vulnerable application so that security-testing traffic could be generated safely.
