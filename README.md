# Notes_do316
Notes_do316

# RBAC

oc get rolebinding -n [project]

oc adm policy add-role-to-user admin [user] -n [project]


# Monitoring

```ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true
```


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
