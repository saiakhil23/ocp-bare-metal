
# Deploying and Configuring Openshift Logging with Persistent Storage 

## Install Cluster Logging Operators
```bash
mkdir logging 

cd logging
```
**1.Create a Namespace for the Elasticsearch Operator**
```bash
vi eo-namespace.yaml 
```

```bash
apiVersion: v1 
kind: Namespace 
metadata:
  name: openshift-operators-redhat
  annotations: 
    openshift.io/node-selector: "" 
  labels: 
    openshift.io/cluster-monitoring: "true"
```
```bash
[igaocpadmin@iga-ocp-ubh01 logging]$ oc create -f eo-namespace.yaml 
namespace/openshift-operators-redhat created
```
**2.Create a Namespace for the Cluster Logging Operator**
```bash
vi clo-namespace.yaml 
```
```bash
apiVersion: v1 
kind: Namespace 
metadata: 
  name: openshift-logging 
  annotations: 
    openshift.io/node-selector: "" 
  labels: 
    openshift.io/cluster-monitoring: "true"
```
```bash
[igaocpadmin@iga-ocp-ubh01 logging]$ oc create -f clo-namespace.yaml 
namespace/openshift-logging created
```
**3.Install the Elasticsearch Operator** 

◦ In the OpenShift Container Platform web console, click Operators → OperatorHub. 

◦ Choose Elasticsearch Operator from the list of available Operators, and click Install. 

◦ Ensure that the All namespaces on the cluster is selected under Installation Mode. 

◦ Ensure that openshift-operators-redhat is selected under Installed Namespace. 

◦ Select Enable operator recommended cluster monitoring on this namespace. 

◦ Select stable as the Update Channel. 

◦ Select an Approval Strategy to Automatic 

◦ Click Install. 

◦ Verify that the Elasticsearch Operator installed by switching to the Operators → Installed 

Operators page. 

◦ Ensure that Elasticsearch Operator is listed in all projects with a Status of Succeeded 

**4.Install the Cluster Logging Operator** 

◦ In the OpenShift Container Platform web console, click Operators → OperatorHub. 

◦ Choose Cluster Logging from the list of available Operators, and click Install. 

◦ Ensure that the A specific namespace on the cluster is selected under Installation Mode. 

◦ Ensure that Operator recommended namespace is openshift-logging under Installed Namespace. 

◦ Select Enable operator recommended cluster monitoring on this namespace. 

◦ Select stable as the Update Channel. 

◦ Select an Approval Strategy to Automatic. 

◦ Click Install. 

◦ Verify that the Cluster Logging Operator installed by switching to the Operators → Installed finish 

◦ Ensure that Cluster Logging is listed in the openshift-logging project with a Status of Succeeded. 

## Create Cluster Logging Instance 
> **Note:** Since we do not have any dynamic provisioing of pvc's wee need to create **PV's** and **StorageClass** separately
> check at the end for pv and sc yaml

• Create a cluster logging instance YAML manifest 

• Create a Cluster Logging instance 
```bash
[igaocpadmin@iga-ocp-ubh01 logging]$ vi clo-instance.yaml 
```
```bash
apiVersion: "logging.openshift.io/v1" 
kind: "ClusterLogging" 
metadata: 
  name: "instance" 
  namespace: "openshift-logging" 
spec: 
  collection: 
    logs: 
      fluentd: 
        resources: 
          limits: 
            cpu: 500m
            memory: 1Gi 
          requests: 
            cpu: 500m 
            memory: 1Gi 
        type: fluentd
  curation:
    curator: 
      resources: 
        limits: 
          memory: 1Gi 
        requests: 
          cpu: 200m 
          memory: 200Mi 
      schedule: 30 3 * * * 
      type: curator 
  logStore: 
    elasticsearch: 
      nodeCount: 3 
      resources: 
        limits: 
          cpu: 2000m
          memory: 16Gi 
        requests: 
          cpu: 1500m 
          memory: 16Gi 
      storage: 
        size: 900Gi 
        storageClassName: elk 
    retentionPolicy: 
      application: 
        maxAge: 7d 
      audit: 
        maxAge: 7d 
      infra: 
        maxAge: 7d 
    type: elasticsearch 
  managementState: Managed 
  visualization: 
    kibana:  
      replicas: 1 
      resources: 
        limits: 
          memory: 1Gi 
        requests: 
          cpu: 500m 
          memory: 1Gi 
    type: kibana
```
**Creating the PVC**
```bash
vi elk-pvc.yaml
```
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: elasticsearch-elasticsearch-cdm-qr26v78k-3
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 500Gi
  storageClassName: elk
```
**Creating the PV**
```bash
vi elk-pv.yaml
```
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elk-pv
spec:
  capacity:
    storage: 200Gi
  accessModes:
  - ReadWriteMany
  nfs:
    path: /home/shares/elk_share
    server: 172.16.3.49
  persistentVolumeReclaimPolicy: Retain
  storageClassName: elk
```
**Creating the storage class**
```bash
vi elk-sc.yaml
```
```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: elk
provisioner: kubernetes.io/no-provisioner
parameters:
  type: nfs
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```
• Create the ClusterLogging instance object as shown below 
```bash
[igaocpadmin@iga-ocp-ubh01 logging]$ oc create -f clo-instance.yaml 
clusterlogging.logging.openshift.io/instance created 
```
• Verify the cluster logging pods deployed in the openshift-logging project 

```bash
[igaocpadmin@iga-ocp-ubh01 logging]$ oc get pods -o wide 

NAME READY STATUS RESTARTS AGE IP NODE NOMINATED NODE READINESS GATES 

cluster-logging-operator-55cbf6cd85-lmsr6 1/1 Running 1 13d 10.64.3.4 iga-ocp-uctl01.goindigo.in <none> <none> 

collector-258sv 2/2 Running 0 9d 10.64.9.16 iga-ocp-uctl01.goindigo.in <none> <none> 

collector-48c8z 2/2 Running 0 9d 10.64.2.22 iga-ocp-uctl02.goindigo.in <none> <none> 

collector-8d8bw 2/2 Running 0 9d 10.64.0.55 iga-ocp-uctl03.goindigo.in <none> <none> 

collector-8tkrw 2/2 Running 0 9d 10.64.8.153 iga-ocp-uaw01.goindigo.in <none> <none> 

collector-92m6r 2/2 Running 2 9d 10.64.3.13 iga-ocp-uaw02.goindigo.in <none> <none> 

collector-dtmhw 2/2 Running 0 9d 10.64.7.21 iga-ocp-uaw03.goindigo.in <none> <none> 

collector-kkwfv 2/2 Running 2 9d 10.64.5.5 iga-ocp-uiw01.goindigo.in <none> <none> 

collector-lwzz2 2/2 Running 0 9d 10.64.1.45 iga-ocp-uiw02.goindigo.in <none> <none> 

collector-mqm2r 2/2 Running 2 9d 10.64.4.21 iga-ocp-uiw03.goindigo.in <none> <none> 

elasticsearch-cdm-g2ncq89l-1-7d75494d88-qkm9c 2/2 Running 0 13d 10.64.7.9 iga-ocp-uiw02.goindigo.in <none> <none> 

elasticsearch-cdm-g2ncq89l-2-b886d96c9-lsfcm 2/2 Running 0 13d 10.64.8.10 iga-ocp-uiw03.goindigo.in <none> <none> 

elasticsearch-cdm-g2ncq89l-3-74c6dc84b-mgrmc 2/2 Running 0 13d 10.64.9.10 iga-ocp-uiw01.goindigo.in <none> <none> 

elasticsearch-im-app-27609780--1-n9vln 0/1 Completed 0 3m50s 10.64.8.100 iga-ocp-uiw02.goindigo.in <none> <none> 

elasticsearch-im-audit-27609780--1-rdp5b 0/1 Completed 0 3m50s 10.64.8.99 iga-ocp-uiw03.goindigo.in <none> <none> 

elasticsearch-im-infra-27609780--1-2l5wn 0/1 Completed 0 3m50s 10.64.8.98 iga-ocp-uiw01.goindigo.in <none> <none> 

kibana-6f44859fd7-kxrnc 2/2 Running 0 13d 10.64.9.12 iga-ocp-uiw03.goindigo.in <none> <none> 
```
• Verify PVCs created for Elasticsearch components 
```bash
[igaocpadmin@iga-ocp-ubh01 logging]$ oc get pvc 

NAME STATUS VOLUME CAPACITY ACCESS MODES STORAGECLASS AGE 

elasticsearch-elasticsearch-cdm-g2ncq89l-1 Bound local-pv-86b45652 900Gi RWO log-mon-thin-csi 13m 

elasticsearch-elasticsearch-cdm-g2ncq89l-2 Bound local-pv-76e747c5 900Gi RWO log-mon-thin-csi 13m 

elasticsearch-elasticsearch-cdm-g2ncq89l-3 Bound local-pv-8c25c0d7 900Gi RWO log-mon-thin-csi 13m 
[igaocpadmin@iga-ocp-ubh01 logging]$ 
```
## **Viewing cluster logs from the Kibana dashboard**

**• To define index patterns and create visualizations in Kibana**

◦ In the OpenShift Container Platform console, click the Application Launcher and select Logging. 

◦ Create your Kibana index patterns by clicking Management → Index Patterns → Create index pattern. 

▪ Users must manually create index patterns to see logs for their projects. Users should create a new index pattern named app and use the @timestamp time field to view their container logs. 

▪ Admin users must create index patterns for the app, infra, and audit indices using the @timestamp time field. 

◦ Create Kibana Visualizations from the new index patterns. 
