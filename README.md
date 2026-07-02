# lab-monitoring

A ready-to-run **Prometheus + Grafana + node_exporter** monitoring stack for a
homelab, deployed with a single `docker compose up -d`. Grafana comes with the
Prometheus datasource and a homelab dashboard already provisioned, plus one
alert rule that fires when any disk exceeds 85% usage.

![Grafana dashboard](screenshots/placeholder-grafana-dashboard.png)

---

## What's included

| Component | Role | URL (default) |
| --- | --- | --- |
| Prometheus | Scrapes metrics, evaluates alert rules | http://localhost:9090 |
| Grafana | Dashboards (auto-provisioned) | http://localhost:3000 |
| node_exporter | Exposes host CPU/RAM/disk/uptime metrics | http://localhost:9100/metrics |

**Provisioned dashboard panels:** CPU usage %, memory used %, disk used % per
mount (with 75%/85% thresholds), and uptime — all filterable by host.

**Alert rule:** `DiskSpaceHigh` — any non-tmpfs filesystem over 85% full for 5
minutes. See [`prometheus/alert.rules.yml`](prometheus/alert.rules.yml).

---

## Requirements

- Docker Engine 24+ and the Docker Compose plugin (`docker compose`, not the
  legacy `docker-compose`).
- Linux host recommended (node_exporter reads the host root filesystem). Works
  on a lab VM.

---

## Setup

```bash
git clone <your-fork-url> lab-monitoring
cd lab-monitoring

# (optional) set a Grafana admin password instead of admin/admin
cp .env.example .env
# edit .env

docker compose up -d
docker compose ps
```

Then open:

- **Grafana** → http://localhost:3000 (login `admin` / `admin` or your `.env`
  values). The **Homelab Overview** dashboard is under the *Homelab* folder.
- **Prometheus** → http://localhost:9090 — check **Status → Targets** (all
  `UP`) and **Alerts** for the `DiskSpaceHigh` rule.

---

## Adding more lab VMs

1. Install node_exporter on the VM (or run the container).
2. Add its address to the `node` job in
   [`prometheus/prometheus.yml`](prometheus/prometheus.yml):

   ```yaml
   - targets: ['192.168.1.20:9100']
     labels: { host: dc01 }
   ```

3. Reload Prometheus:

   ```bash
   curl -X POST http://localhost:9090/-/reload
   ```

The dashboard's `host` variable will pick up the new target automatically.

---

## Testing the alert

Fill a disk on a monitored host past 85% (e.g. `fallocate -l 5G /tmp/fill`),
wait ~5 minutes, and watch **Prometheus → Alerts** flip the `DiskSpaceHigh`
rule to *Firing*. Remove the file to clear it. Wire it to email/Slack later by
adding Alertmanager.

---

## Teardown

```bash
docker compose down          # stop, keep data volumes
docker compose down -v       # stop and delete Prometheus/Grafana data
```

## License

MIT — see [LICENSE](LICENSE).
