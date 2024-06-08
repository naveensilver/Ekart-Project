






++++++++++++++++++++++++++

## Setting up the Instances and Security Groups

Create individual machines for the Jenkins, Nexus, and Sonarqube [Setting 1 machine for each tool used]

Machine Details: t2 medium; 20 GB Storage; Ubuntu 20.04

For Security Groups [Add the following groups]

```
SMTP TCP 25
Custom TCP - TCP Range from 30000 to 32767 // For Kubernetes NodePOrt Service
Custom TCP - TCP Range from 3000 to 10000 // Maximum Installations of Tools under these Ranges
HTTP - TCP - 80
HTTPS - TCP - 443
SSH TCP 22
CUSTOM TCP 6443 //  the Kubernetes API server listens on port 6443 on the first non-localhost network interface, protected by TLS
```
