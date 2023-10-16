# Notes_do316
Notes_do316


# Network

oc expose service/static --path=/static --hostname=[hostname]****

oc edit vm [virtual-machine] -n [namespace]


```ǹetwork
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: bridge-dev  
  annotations:
    k8s.v1.cni.cncf.io/resourceName: bridge.network.kubevirt.io/bridge-dev 
spec:
  config: '{
    "cniVersion": "0.3.1",
    "name": "bridge-dev",  
    "type": "cnv-bridge",  
    "bridge": "dev-bridge", 
    "macspoofchk": true, 
    "vlan": 0 
  }
```
