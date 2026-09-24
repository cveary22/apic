# API Connect Monitoring 

## What monitoring can API Connect do? 

API Connect has a handful of Rest APIs and health checks via OCP cli that can be utilized to monitor the status of the APIC components. There are several support documents outlining the commands and REST APIs 
 

### OpenShift CLI commands 

Since the deployment is CP4I\Top-level CR to check the status of the cluster, run the following 
 
```oc get apiconnectcluster -n <APIC_namespace>``` 

Ref: https://www.ibm.com/docs/en/api-connect/10.0.8?topic=checks-monitoring-cloud-pak-integration  
 

You can run the command similarly to obtain the status of individual subsystems 
Commands: 
| Management	|	Portal	|	Analytics	|	Gateway | 
|---|---|---|---|
| ```oc get mgmt``` |	```oc get ptl``` |	```oc get a7s``` | ```oc gw``` | 

Ref: https://www.ibm.com/docs/en/api-connect/10.0.8?topic=openshift-monitoring-api-connect-cluster  


### REST APIS 

API Rest calls to consider calling regularly  

**Analytics** 
Make a REST API call like this (with a cloud level authentication token): 
```GET /analytics/{analytics-service}/cloud/service-status``` 

or via the CLI (having done a cloud level apic login): 
```apic -m analytics service:cloudServicestatus --server <platform API host> --analytics-service <analytics service name> --format json``` 

Ref: https://community.ibm.com/community/user/integration/blogs/chris-dudley1/2024/09/01/monitoring-apic-analytics  

**Portal Sites** 

```site_url/health```
 

Ref: https://www.ibm.com/docs/en/api-connect/10.0.8?topic=mhc-obtaining-simple-health-check-data-developer-portal-sites-by-using-rest-api-call 

 

## OpenShift monitoring Alerts 

Things to consider 

+ We have various APIs that can be used to build on the basics they provide, but let the monitoring solution monitor for things like pod restarts, pod downtime, etc. 
 
+ For stateless pods like apim or lur a single pod restart is not that big a deal if you are running three of them. You might be interested if a pod is down and stays down for an extended period, or if something is restarting frequently. 
 
+ Consider alerts on the number of ready replicas for a replicaset rather than individual pods. 
 
+ Our experience with gateways is that restarts can cause unexpected issues, so they deserve more attention 
 
+ Monitor the gateway pod memory and CPU usage separately 
 
+ **Be careful in things like monitoring memory as, eg analytics storage will use 90% of assigned memory the second it starts** 
 
+ Configure alerts for available disk space – several problems are expected if you run out  
 
+ Monitoring CPU won’t provide too much valuable insight 
 
+ Allow the system to record those values (CPU/memory) for a few weeks of normal usage before assigning alert thresholds so you know what “normal” is 
 
+ There is no general monitoring for APIC deployment profiles - each pod has limits that can be monitored over time for overall usage (eg. CPU/memory/storage). When pods are nearing their limit, upgrading to the next profile size is recommended.  

 

 

### Critical APIC pods to monitor: 

| Pod | Description | Notes | 
| --- | ---| ---| 
|Db (mgmt. pod) |Postgres database pods - PostgreSQL serves as the backend database, storing crucial metadata and configuration details for APIs, products, and applications. | |
| Storage (a7s pod) | The storage pods contain the OpenSearch database that stores all the analytics data. By default, the storage pods also run the OpenSearch cluster management tasks. If dedicated storage is enabled the OpenSearch cluster management tasks are done by the storage-os-master pods, and the storage pods contain just the analytics data. | **Be careful monitoring for memory: analytics storage will use 90% of assigned memory the second it starts**  |
| portal db (portal) | This is the database pod which has 2 running containers. • portal db-dbproxy container • portal db-db container These containers are explained below. | | 
| portal db-db container | Hosts the Portal Databases. | |
| portal db-dbproxy container | Handles communication with portal db container from the portal www pod. | |
| portal www | This pod hosts the portal sites and contains the admin and web containers. | |
| datapower (gwy) | The DataPower runtime instance | Our experience with gateways is that restarts can cause unexpected issues, so they deserve more attention <br/> <br/> Monitor the gateway pod memory and CPU usage separately |

 
**Optional** 

| Pod | Description | Notes | 
| --- | ---| ---| 
| Apim| The core microservice in the API Management subsystem. It handles communication with the other subsystems and is the backend for the UI. Any action taken in API Connect (logging in, publishing a product, creating a Catalog) is coordinated through this microservice. A standard (HA) deployment has at least three pods, deployed across the different nodes. This number of pods can be scaled up based on the load. | |
| taskmanager | The Task Manger manages and regulate internal tasks in API Connect. | |
| storage-os-master (a7s) | When analytics is configured to use dedicated storage, this is used to manage the storage. | |
| portal nginx | This pod runs on a single container. It is a simple proxy for communication. | |

 
### Key Pods in terms of Performance
**Manager**
+ Task manager
+ Postgres
+ Apim

**Portal** 
+ Admin
+ Db
+ Nginx 

**Analytics**
+ Director
+ Ingestion
+ Mtls-gw
+ storage

**Gateway**
+ gwv6


 

 

### Other Monitoring solutions  

**API Connect Trawler**
https://github.com/IBM/apiconnect-trawler 

**Instana**
https://www.ibm.com/docs/en/instana-observability/current?topic=technologies-monitoring-api-connect  

 
