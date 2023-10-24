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

Virtual Machone

```virtualMachine
template:
    metadata:
      labels:
        app: [app-label]
```

Service

```service
apiVersion: v1
kind: Service
metadata:
  name: [app-label]
  namespace: [namespace]
spec:
  type: ClusterIP
  selector:
    app: [app-label]
  ports:
    - protocol: TCP
      port: [port]
      targetPort: [targetport]
```

NetworkPolicy

```service
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: [np-name]
spec:
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          app: [app-label]
    ports:
    - port: [port]
      protocol: TCP
  podSelector: {}
```

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
# Node selector

Node Selector
```
metadata:
  name: example-vm-node-selector
apiVersion: kubevirt.io/v1
kind: VirtualMachine
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: example.io/example-key
            operator: In
            values:
            - example-value-1
            - example-value-2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: example-node-label-key
            operator: In
            values:
            - example-node-label-value
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "virtualization"
    effect: "NoSchedule"
  template:
    spec:
      nodeSelector:
        example-key-1: example-value-1
        example-key-2: example-value-2
...output omitted...
