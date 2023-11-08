# Creating NTP server
Install NTP packages on both client(master and worker) and server machines(for us it is on bastion)
```
dnf install chrony*
```
## Edit the /etc/chrony.conf in SERVER machine
- under the Allow NTP client access from local networkadd the clients machines that you wan to add
```
# Allow NTP client access from local network.
allow 172.16.3.56/25
allow 172.16.3.57/25
allow 172.16.3.58/25
```
## For the clients(master and worker)
- create chrony.conf file in bastion and encode with base64
```
 vi chrony.conf
```
172.16.3.49 is our nfs server
```
server 172.16.3.49 iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```
- crreate a yaml file for client machines (We use MachineConfig for both master and worker chrony.conf editing )
```
vi ntp_master.yaml
```
- Edit the ##data## parameter in the ntp_master.yaml with base64 chrony.conf
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: master
  name: master-chrony-config
spec:
  config:
    ignition:
      config: {}
      security:
        tls: {}
      timeouts: {}
      version: 3.2.0
    networkd: {}
    passwd: {}
    storage:
      files:
      - contents:
              source: data:text/plain;charset=utf-8;base64,c2VydmVyIDE3Mi4xOS4xOTAuMzcgaWJ1cnN0CnNlcnZlciAxNzIuMTkuMTkwLjM4IGlidXJzdApzZXJ2ZXIgMTcyLjE3LjEzMS4xMCBpYnVyc3QKZHJpZnRmaWxlIC92YXIvbGliL2Nocm9ueS9kcmlmdAptYWtlc3RlcCAxLjAgMwpydGNzeW5jCmxvZ2RpciAvdmFyL2xvZy9jaHJvbnkKCg==
        mode: 420
        filesystem: root
        overwrite: true
        path: /etc/chrony.conf
  osImageURL: ""
```
- Apply the above yaml to apply the changes in master node
```
oc apply -f ntp_master.yaml
```
- create NTP yaml for worker nodes
```
vi ntp_worker.yaml
```
- Edit the DATA parameter in the yaml below yaml file for worker nodes
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: worker
  name: worker-chrony-config
spec:
  config:
    ignition:
      config: {}
      security:
        tls: {}
      timeouts: {}
      version: 3.2.0
    networkd: {}
    passwd: {}
    storage:
      files:
      - contents:
              source: data:text/plain;charset=utf-8;base64,c2VydmVyIDE3Mi4xOS4xOTAuMzcgaWJ1cnN0CnNlcnZlciAxNzIuMTkuMTkwLjM4IGlidXJzdApzZXJ2ZXIgMTcyLjE3LjEzMS4xMCBpYnVyc3QKZHJpZnRmaWxlIC92YXIvbGliL2Nocm9ueS9kcmlmdAptYWtlc3RlcCAxLjAgMwpydGNzeW5jCmxvZ2RpciAvdmFyL2xvZy9jaHJvbnkKCg==
        mode: 420
        filesystem: root
        overwrite: true
        path: /etc/chrony.conf
  osImageURL: ""
```
```
oc apply -f ntp_worker.yaml
```
