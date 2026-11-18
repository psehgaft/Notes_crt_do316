# Configure Health Probes for Virtual Machines

> Very short, YAML-first summary. Domain in all examples: **psehgaft.org**.

**Objective:** Add probes to VMs behind a Service so Kubernetes detects app availability and updates Service networking — or restarts the VM.

## At a glance

| Check | Question it answers | On failure | Detects |
|---|---|---|---|
| `readinessProbe` | Ready to serve traffic? | VM IP removed from Service endpoints (VM keeps running) | Slow start, cache warm-up, long batch jobs, temporary loss |
| `livenessProbe` | Healthy, or stuck? | VMI deleted and redeployed | Hung app that needs a full VM restart |
| `watchdog` (device) | Is the OS alive? | `poweroff` / `reset` / `shutdown` | Hung OS only — **not** app failures |

Probe methods: `httpGet` (app returns `200-399` on the probe path), `tcpSocket` (TCP connect succeeds), `guest agent ping` (QEMU guest agent — Technology Preview, out of scope).

Common timing fields: `initialDelaySeconds` (wait before first check), `periodSeconds` (frequency, default `10`), `timeoutSeconds` (default `1`), `failureThreshold` (default `3`), `successThreshold` (default `1`).

## 1. Readiness — `httpGet` — [`01-readiness-probe-httpget.yaml`](01-readiness-probe-httpget.yaml)

Web VM `www1` on `8080`, endpoint `/health`:

```yaml
spec:
  template:
    spec:
      readinessProbe:
        httpGet:
          path: /health
          port: 8080
          host: www1.psehgaft.org
        initialDelaySeconds: 120
        periodSeconds: 20
        timeoutSeconds: 10
        failureThreshold: 3
        successThreshold: 3
```

## 2. Liveness — `tcpSocket` — [`02-liveness-probe-tcpsocket.yaml`](02-liveness-probe-tcpsocket.yaml)

DB VM `mariadb-server` on `3306` (no HTTP, so TCP):

```yaml
spec:
  template:
    spec:
      livenessProbe:
        tcpSocket:
          port: 3306
          host: mariadb.psehgaft.org
        initialDelaySeconds: 120
        periodSeconds: 20
```

## 3. Watchdog — [`03-watchdog.yaml`](03-watchdog.yaml)

Only when no probe can test the app. OS must support Intel 6300ESB:

```yaml
spec:
  template:
    spec:
      domain:
        devices:
          watchdog:
            name: mywatchdog
            i6300esb:
              action: poweroff   # poweroff | reset | shutdown
```

## Apply

```sh
oc apply -f 01-readiness-probe-httpget.yaml
oc apply -f 02-liveness-probe-tcpsocket.yaml
oc apply -f 03-watchdog.yaml
# Or edit live: oc edit vm www1 / oc edit vm mariadb-server
```

Console: **Virtualization → VirtualMachines → [VM] → YAML → Save**. If the VM is already running, **restart it** to apply probe changes.

## References

- OpenShift 4.18 Virtualization — VM health checks: https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/virtualization/index#virt-monitoring-vm-health
- OpenShift 4.18 Building Applications — Application health: https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/building_applications/index#application-health
- OpenShift 4.18 Virtualization — Watchdog: https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html-single/virtualization/index#watchdog_virt-monitoring-vm-health
