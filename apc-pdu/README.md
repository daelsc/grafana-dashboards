# APC PDU Monitoring (Prometheus + Grafana)

Monitor APC Metered Rack PDUs (AP8866 / rPDU2G) via SNMP, with Prometheus scraping and a Grafana dashboard.

Supports multiple daisy-chained PDUs through a single NMC (Network Management Card).

## Files

| File | Purpose |
|------|---------|
| `snmp-exporter-apc.yml` | SNMP exporter config with APC rPDU2 OID mappings |
| `apc-pdu-dashboard.json` | Grafana dashboard (importable) |

## Metrics Collected

**Per PDU:** total power (kW), peak power, energy (kWh), load %

**Per Phase:** current (A), voltage (V), power (kW), apparent power (kVA), power factor

**Per Bank:** current (A), peak current

## Setup

### 1. SNMP Exporter

Copy `snmp-exporter-apc.yml` to your SNMP exporter config path and update the `auths` section with your community strings:

```yaml
auths:
  apc_public:
    community: your_community_string
    version: 2
```

Run the exporter (Docker example):

```bash
docker run -d --name snmp_exporter \
  -p 9116:9116 \
  -v /path/to/snmp-exporter-apc.yml:/etc/snmp_exporter/snmp.yml:ro \
  --restart unless-stopped \
  prom/snmp-exporter:latest
```

### 2. Prometheus

Add a scrape job targeting your PDU IPs:

```yaml
- job_name: 'snmp-apc'
  metrics_path: /snmp
  static_configs:
    - targets:
        - 10.1.115.241   # your PDU IP(s)
  params:
    module: [apc_rpdu2_nmc2]
    auth: [apc_public]
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: localhost:9116   # SNMP exporter address
```

### 3. Grafana Dashboard

Import `apc-pdu-dashboard.json`:

1. Go to **Dashboards > Import**
2. Upload `apc-pdu-dashboard.json` or paste its contents
3. Select your Prometheus datasource
4. Click **Import**

The dashboard includes a PDU dropdown selector for multi-PDU setups.

## APC PDU SNMP Requirements

- SNMP must be enabled on the PDU's NMC
- Add your Prometheus server's IP as an NMS host in the PDU's SNMP configuration
- Tested with NMC2 firmware v7.2.x and AP8866 (should work with other rPDU2G models)

## Unit Reference

| SNMP Metric | Raw Unit | Display |
|---|---|---|
| Power (`_hundredths_kw`) | hundredths of kW | / 100 = kW |
| Energy (`_tenths_kwh`) | tenths of kWh | / 10 = kWh |
| Current (`_tenths_amp`) | tenths of A | / 10 = A |
| Voltage | volts | raw |
| Power factor | percent | raw |
