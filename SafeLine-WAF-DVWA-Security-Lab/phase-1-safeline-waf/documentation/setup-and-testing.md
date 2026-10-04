# Phase 1 — SafeLine WAF + DVWA Setup and Testing

## 1. Lab Goal

The first phase focused on understanding web application security monitoring and the role of a Web Application Firewall.

The target architecture was:

```text
Kali Linux → SafeLine WAF → DVWA / Apache
```

## 2. Environment

The lab used:

- Ubuntu
- Kali Linux
- Docker
- SafeLine WAF
- Apache
- DVWA

## 3. DVWA Deployment

DVWA was deployed on Ubuntu using Apache as the web server.

DVWA was intentionally used as a vulnerable web application so that controlled security-testing traffic could be generated from Kali Linux.

After deployment, the application was verified from the lab environment.

## 4. SafeLine Deployment

SafeLine WAF was deployed using Docker.

SafeLine was configured as a reverse proxy in front of the DVWA application.

The resulting traffic path was:

```text
Kali / Client
      ↓
SafeLine WAF
      ↓
DVWA / Apache
```

## 5. Security Testing

Kali Linux was used to generate controlled web security-testing traffic against DVWA.

The purpose was to observe how SafeLine identifies and records potentially malicious web requests.

Testing was performed only against the intentionally vulnerable lab environment.

## 6. SafeLine Detection

SafeLine successfully detected and recorded web security events.

The detection information included fields such as:

- Source IP
- Requested URL
- Attack type
- Risk level
- Triggered WAF rule

## 7. Validation

The first phase was considered successful when:

- DVWA was reachable
- SafeLine operated in front of DVWA
- Test traffic passed through SafeLine
- SafeLine generated security detection records
- Detection information was visible in the SafeLine environment

## 8. Phase 1 Result

The completed phase established a working web security monitoring environment:

```text
Kali
 ↓
SafeLine WAF
 ↓
DVWA / Apache
```

SafeLine successfully detected and recorded the security-testing activity.

The next phase extended these detections into Wazuh SIEM.
