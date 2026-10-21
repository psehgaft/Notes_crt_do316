# Study Notes — OpenShift Virtualization (DO316 Track)

Personal study notes for managing virtual machines with Red Hat OpenShift Virtualization (KubeVirt): RBAC, user-workload monitoring, networking, node placement, and VM health probes.

> [!IMPORTANT]
> This is an independent study repository. It is not an official Red Hat course, exam guide, or exam dump. Run privileged commands only against a disposable lab cluster you are authorized to administer.

## Table of contents

- [RBAC](#rbac)
- [Monitoring](#monitoring)
- [Network](#network)
- [Node placement](#node-placement)
- [Probes](#probes)

## RBAC

```sh
oc get rolebinding -n [project]

oc adm policy add-role-to-user admin [user] -n [project]
```

## Monitoring

Enable user workload monitoring by editing the cluster-monitoring configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true
```

## Network

VirtualMachine template:

```yaml
template:
  metadata:
    labels:
      app: [app-label]
```

Service:

```yaml
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

NetworkPolicy:

```yaml
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

Expose and edit virtual machines:

```sh
oc expose service/static --path=/static --hostname=[hostname]

oc edit vm [virtual-machine] -n [namespace]
```

NetworkAttachmentDefinition (KubeVirt bridged networking):

```yaml
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
  }'
```

## Node placement

Node affinity, tolerations, node selectors, and migration strategies:

```yaml
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
      evictionStrategy: LiveMigrate
      runStrategy: RerunOnFailure
      nodeSelector:
        example-key-1: example-value-1
        example-key-2: example-value-2
```

## Probes

VM health probes (`readinessProbe` / `livenessProbe`):

```yaml
metadata:
  name: example-vm-node-selector
apiVersion: kubevirt.io/v1
kind: VirtualMachine
spec:
  template:
    spec:
      domain:
        readinessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 2
          failureThreshold: 2
          successThreshold: 1
        livenessProbe:
          tcpSocket:
            port: [port]
          initialDelaySeconds: 10
          periodSeconds: 5
        evictionStrategy: LiveMigrate
```
