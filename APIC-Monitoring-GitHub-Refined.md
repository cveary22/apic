# API Connect Monitoring

A reference guide for monitoring IBM API Connect components, subsystem health, pod availability, resource usage, and supporting monitoring solutions.

> [!NOTE]
> This document is intended to be maintained as a living reference. Monitoring thresholds should be based on observed normal usage in the target environment.

## Table of Contents

- [Quick Reference](#quick-reference)
- [OpenShift CLI Health Checks](#openshift-cli-health-checks)
- [REST API Health Checks](#rest-api-health-checks)
- [Alerting Guidance](#alerting-guidance)
- [Monitoring Priority Matrix](#monitoring-priority-matrix)
- [Critical APIC Pods to Monitor](#critical-apic-pods-to-monitor)
- [Optional Pods to Monitor](#optional-pods-to-monitor)
- [Key Pods in Terms of Performance](#key-pods-in-terms-of-performance)
- [Known Monitoring Pitfalls](#known-monitoring-pitfalls)
- [Other Monitoring Solutions](#other-monitoring-solutions)
- [References](#references)

---

## Quick Reference

### Cluster Health

```bash
oc get apiconnectcluster -n <APIC_namespace>
```

### Individual Subsystem Status

| Subsystem | Command |
|---|---|
| Management | `oc get mgmt` |
| Portal | `oc get ptl` |
| Analytics | `oc get a7s` |
| Gateway | `oc gw` |

### Key Health Checks

#### Analytics

```bash
apic -m analytics service:cloudServicestatus \
  --server <platform API host> \
  --analytics-service <analytics service name> \
  --format json
```

#### Portal Site

```text
site_url/health
```

### Most Important Monitoring Recommendations

- Monitor for pod restarts and pod downtime.
- Consider alerts on the number of ready replicas for a ReplicaSet rather than on individual pods.
- Give gateway restarts additional attention because they can cause unexpected issues.
- Monitor gateway pod memory and CPU usage separately.
- Configure alerts for available disk space because several problems are expected if you run out.
- Record CPU and memory values for a few weeks of normal usage before assigning alert thresholds.
- Monitor pod CPU, memory, and storage limits over time. When pods are nearing their limits, upgrading to the next profile size is recommended.

> [!WARNING]
> Be careful when monitoring Analytics storage memory. Analytics storage will use 90% of assigned memory the second it starts.

---

## OpenShift CLI Health Checks

API Connect has a handful of Rest APIs and health checks via OCP cli that can be utilized to monitor the status of the APIC components. There are several support documents outlining the commands and REST APIs.

### Top-Level API Connect Cluster

Since the deployment is CP4I\Top-level CR to check the status of the cluster, run the following:

```bash
oc get apiconnectcluster -n <APIC_namespace>
```

### Individual Subsystems

You can run commands similarly to obtain the status of individual subsystems.

<details>
<summary><strong>Management</strong></summary>

```bash
oc get mgmt
```

</details>

<details>
<summary><strong>Portal</strong></summary>

```bash
oc get ptl
```

</details>

<details>
<summary><strong>Analytics</strong></summary>

```bash
oc get a7s
```

</details>

<details>
<summary><strong>Gateway</strong></summary>

```bash
oc gw
```

</details>

---

## REST API Health Checks

API Rest calls to consider calling regularly.

### Analytics

Make a REST API call like this with a cloud level authentication token, or via the CLI after completing a cloud level `apic` login:

```bash
apic -m analytics service:cloudServicestatus \
  --server <platform API host> \
  --analytics-service <analytics service name> \
  --format json
```

### Portal Sites

```text
site_url/health
```

---

## Alerting Guidance

### Availability

- Let the monitoring solution monitor for things like pod restarts and pod downtime.
- For stateless pods like apim or lur, a single pod restart is not that big a deal if you are running three of them.
- You might be interested if a pod is down and stays down for an extended period, or if something is restarting frequently.
- Consider alerts on the number of ready replicas for a ReplicaSet rather than individual pods.

### Gateway-Specific Monitoring

> [!IMPORTANT]
> Our experience with gateways is that restarts can cause unexpected issues, so they deserve more attention.

- Monitor gateway restarts.
- Monitor the gateway pod memory and CPU usage separately.

### Capacity and Resources

- Configure alerts for available disk space. Several problems are expected if you run out.
- Monitoring CPU won’t provide too much valuable insight.
- Allow the system to record CPU and memory values for a few weeks of normal usage before assigning alert thresholds so you know what “normal” is.
- There is no general monitoring for APIC deployment profiles. Each pod has limits that can be monitored over time for overall usage, such as CPU, memory, and storage.
- When pods are nearing their limits, upgrading to the next profile size is recommended.

### Analytics Storage Memory

> [!WARNING]
> Be careful in things like monitoring memory as, eg analytics storage will use 90% of assigned memory the second it starts.

---

## Monitoring Priority Matrix

This matrix highlights the monitoring emphasis described in the source notes. It does not assign new alert thresholds.

| Component | Monitoring Emphasis | Monitor For |
|---|---|---|
| DataPower Gateway | Special attention | Restarts, memory, and CPU usage |
| Analytics Storage | Special consideration | Available disk space and expected high memory usage |
| Management Db | Critical pod | Availability |
| Portal Db | Critical pod | Availability |
| Portal WWW | Critical pod | Availability |
| Apim | Optional pod | Availability and repeated restarts |
| taskmanager | Optional pod | Availability |
| storage-os-master | Optional pod | Availability when dedicated storage is enabled |
| portal nginx | Optional pod | Availability |

---

## Critical APIC Pods to Monitor

<details>
<summary><strong>Db (mgmt. pod)</strong></summary>

### Description

Postgres database pods. PostgreSQL serves as the backend database, storing crucial metadata and configuration details for APIs, products, and applications.

</details>

<details>
<summary><strong>Storage (a7s pod)</strong></summary>

### Description

The storage pods contain the OpenSearch database that stores all the analytics data.

By default, the storage pods also run the OpenSearch cluster management tasks. If dedicated storage is enabled, the OpenSearch cluster management tasks are done by the storage-os-master pods, and the storage pods contain just the analytics data.

### Monitoring Note

> [!WARNING]
> Be careful monitoring for memory: analytics storage will use 90% of assigned memory the second it starts.

</details>

<details>
<summary><strong>portal db (portal)</strong></summary>

### Description

This is the database pod which has 2 running containers:

- portal db-dbproxy container
- portal db-db container

</details>

<details>
<summary><strong>portal db-db container</strong></summary>

### Description

Hosts the Portal Databases.

</details>

<details>
<summary><strong>portal db-dbproxy container</strong></summary>

### Description

Handles communication with portal db container from the portal www pod.

</details>

<details>
<summary><strong>portal www</strong></summary>

### Description

This pod hosts the portal sites and contains the admin and web containers.

</details>

<details>
<summary><strong>datapower (gwy)</strong></summary>

### Description

The DataPower runtime instance.

### Monitoring Notes

- Our experience with gateways is that restarts can cause unexpected issues, so they deserve more attention.
- Monitor the gateway pod memory and CPU usage separately.

</details>

---

## Optional Pods to Monitor

<details>
<summary><strong>Apim</strong></summary>

The core microservice in the API Management subsystem. It handles communication with the other subsystems and is the backend for the UI.

Any action taken in API Connect, including logging in, publishing a product, or creating a Catalog, is coordinated through this microservice.

A standard (HA) deployment has at least three pods, deployed across the different nodes. This number of pods can be scaled up based on the load.

</details>

<details>
<summary><strong>taskmanager</strong></summary>

The Task Manger manages and regulate internal tasks in API Connect.

</details>

<details>
<summary><strong>storage-os-master (a7s)</strong></summary>

When analytics is configured to use dedicated storage, this is used to manage the storage.

</details>

<details>
<summary><strong>portal nginx</strong></summary>

This pod runs on a single container. It is a simple proxy for communication.

</details>

---

## Key Pods in Terms of Performance

| Subsystem | Key Pods |
|---|---|
| Manager | Task manager, Postgres, Apim |
| Portal | Admin, Db, Nginx |
| Analytics | Director, Ingestion, Mtls-gw, storage |
| Gateway | gwv6 |

---

## Known Monitoring Pitfalls

### Analytics Storage Memory

Analytics storage will use 90% of assigned memory the second it starts. Be careful when defining memory alerts for Analytics storage.

### Stateless Pod Restarts

For stateless pods like apim or lur, a single pod restart is not that big a deal if you are running three of them. A pod that stays down for an extended period or restarts frequently may be more relevant.

### Individual Pods Versus ReplicaSets

Consider alerts on the number of ready replicas for a ReplicaSet rather than individual pods.

### CPU Monitoring

Monitoring CPU won’t provide too much valuable insight. Allow the system to record CPU and memory values for a few weeks of normal usage before assigning alert thresholds so you know what “normal” is.

### Available Disk Space

Configure alerts for available disk space. Several problems are expected if you run out.

### Deployment Profiles

There is no general monitoring for APIC deployment profiles. Each pod has limits that can be monitored over time for overall usage, such as CPU, memory, and storage. When pods are nearing their limits, upgrading to the next profile size is recommended.

---

## Other Monitoring Solutions

### API Connect Trawler

- [API Connect Trawler on GitHub](https://github.com/IBM/apiconnect-trawler)

### Instana

- [Monitoring API Connect with Instana](https://www.ibm.com/docs/en/instana-observability/current?topic=technologies-monitoring-api-connect)

---

## References

### IBM Documentation

- [Monitoring Cloud Pak for Integration](https://www.ibm.com/docs/en/api-connect/10.0.8?topic=checks-monitoring-cloud-pak-integration)
- [Monitoring the API Connect cluster on OpenShift](https://www.ibm.com/docs/en/api-connect/10.0.8?topic=openshift-monitoring-api-connect-cluster)
- [Obtaining simple health check data for Developer Portal sites by using a REST API call](https://www.ibm.com/docs/en/api-connect/10.0.8?topic=mhc-obtaining-simple-health-check-data-developer-portal-sites-by-using-rest-api-call)

### Community Resources

- [Monitoring APIC Analytics](https://community.ibm.com/community/user/integration/blogs/chris-dudley1/2024/09/01/monitoring-apic-analytics)
- [API Connect Trawler on GitHub](https://github.com/IBM/apiconnect-trawler)

### Monitoring Platforms

- [Monitoring API Connect with Instana](https://www.ibm.com/docs/en/instana-observability/current?topic=technologies-monitoring-api-connect)
