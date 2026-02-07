# Grafana Dashboards

Grafana dashboards for cluster monitoring. Each subdirectory contains an importable dashboard JSON and any supporting configs.

## Dashboards

| Dashboard | Description |
|-----------|-------------|
| [apc-pdu](apc-pdu/) | APC Metered Rack PDU monitoring via SNMP (AP8866 / rPDU2G) |

## Importing

1. In Grafana, go to **Dashboards > Import**
2. Upload the `*-dashboard.json` file from the relevant subdirectory
3. Select your Prometheus datasource when prompted
4. Click **Import**
