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
- Restart the server chronyd service
```
systemctl enable chronyd
systemctl restart chronyd
```
## For the clients(master and worker)
- create chrony.conf file in bastion and encode with base64
```
 vi chrony_client.conf
```
172.16.3.49 is our nfs server
```
server 172.16.3.49 iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
```
172.16.3.49 is our nfs server

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
# Things to do after NTP server and client configuration
1. first check the server(bastion) can detect the clients
```
chronyc clients
```
output 
```
Hostname                      NTP   Drop Int IntL Last     Cmd   Drop Int  Last
===============================================================================
master2.ocp4.imss.com          11      0   6   -    5d       0      0   -     -
master0.ocp4.imss.com          59      0   6   -    5d       0      0   -     -
localhost                       0      0   -   -     -       1      0   -    18

has context menu
```
2. check the clients is pointing to master or not
```
chronyc sources
````
output should point to NTP server not to INTERNET, below o/p showing the internet
````
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^+ time.cloudflare.com           3  10   377    45  -3966us[-3966us] +/-   65ms
^* 172-232-97-196.ip.linode>     2  10   377   271  +2241us[+2483us] +/-   26ms
^- ntp2.ggsrv.de                 2  10   377   918    +15ms[  +15ms] +/-   98ms
^+ ntp5.mum-in.hosts.301-mo>     2  10   377   991  -2230us[-1980us] +/-   66ms
```
3. check the sync and active status of NTP service
```
sudo timedatectl
```
OUTPUT
```
               Local time: Wed 2023-11-08 16:21:37 IST
           Universal time: Wed 2023-11-08 10:51:37 UTC
                 RTC time: Wed 2023-11-08 10:51:37
                Time zone: Asia/Kolkata (IST, +0530)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no

```
