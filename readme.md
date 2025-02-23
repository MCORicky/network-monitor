# Network Monitoring Stack

This repository contains a comprehensive network monitoring stack built with Docker Compose. The stack aggregates metrics, logs, and network device data into one unified solution. It includes:

- **Prometheus:** For metrics collection.
- **Grafana:** For visualization and dashboards.
- **Loki:** For log aggregation.
- **Promtail:** For log shipping.
- **SNMP Exporter:** For collecting SNMP metrics from network devices.
- **Dashboard:** A simple static webpage listing service URLs for quick access.

All services are containerized, and the stack is designed for deployment on an on-prem Docker host (referred to internally as `docker-host`).

## Prerequisites

- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/install/)
- SNMP-enabled network devices (if you plan to use the SNMP Exporter)
- (Optional) GitHub Actions for CI/CD deployment

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/MCORicky/network-monitor.git
cd network-monitor
```
### 2. Configure Your Environment
Prometheus:
Edit prometheus/prometheus.yml to add additional scrape targets or adjust intervals.

Loki:
Edit loki/local-config.yaml to customize storage, retention, and compactor settings.

Promtail:
Update promtail/promtail-config.yaml to change the log paths you wish to monitor.

SNMP Exporter:
Modify snmp_exporter/snmp.yml with the SNMP modules and metrics relevant to your network devices.

Dashboard:
Customize dashboard/index.html to add URLs, descriptions, or additional hosts as needed.

### 3. Prepare Host Directories (for Loki)
On your Docker host (or within the deployment directory), ensure the following directories exist and have the proper permissions (adjust UID if necessary):

```bash
mkdir -p loki/index loki/cache loki/chunks loki/wal loki/compactor
sudo chown -R 10001:10001 loki/index loki/cache loki/chunks loki/wal loki/compactor
```
### 4. Deploy the Stack
Run the following command to build and start all services:

```bash
docker-compose up -d --build
```

### 5. Verify the Deployment
Prometheus:
Open http://docker-host:9090 in your browser. Check the "Targets" page under Status to verify that Prometheus is scraping metrics.

Grafana:
Open http://docker-host:3000 and log in (default credentials are typically admin/admin on first login; you'll be prompted to change your password).
Configure Prometheus and Loki as data sources, then create or import dashboards.

Loki & Promtail:
Use Grafana's Explore section to query logs from the Loki data source, verifying that Promtail is shipping logs correctly.

SNMP Exporter:
Ensure the SNMP Exporter is running on port 9116, and verify in Prometheus (via its "Targets" page) that SNMP metrics are being scraped.

Dashboard:
Open http://docker-host:8080 to view the static webpage with links to your services and network hosts.

CI/CD Deployment
This repository is designed for integration with CI/CD pipelines (e.g., GitHub Actions). A sample deployment workflow is included in .github/workflows/deploy.yml. This workflow:

```yaml
Pulls the latest changes from GitHub.
Rebuilds and deploys the Docker Compose stack (including the dashboard) on your on-prem server via SSH.
- name: Deploy via SSH
  uses: appleboy/ssh-action@v0.1.5
  with:
    host: docker-host  # or your public domain if applicable
    username: ricky
    port: 22
    key: ${{ secrets.SERVER_SSH_KEY }}
    script: |
      cd /home/ricky/monitoring-stack
      if [ -d ".git" ]; then
        git pull origin main
      else
        git clone https://${{ secrets.GH_TOKEN }}@github.com/MCORicky/network-monitor.git .
      fi
      docker-compose up -d --build
```
Troubleshooting
Container Logs:
Use docker-compose logs <service> (e.g., docker-compose logs loki) to diagnose issues.

Directory Permissions:
Ensure that host directories mapped to Loki have the correct ownership and permissions.

Service Configurations:
Validate that configuration files (e.g., local-config.yaml, prometheus.yml) are properly formatted and contain the necessary settings.

Contributing
Contributions, bug reports, and feature requests are not considered at this time.