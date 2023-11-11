# CKS

### Network policies

Deny all except DNS as default.

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  - Ingress
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
```

Allow connection only between allowed selectors.

#### Close node metadata access (GCloud, AKS, AWS) 

```
# all pods in namespace cannot access metadata endpoint
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cloud-metadata-deny
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0
        except:
        - 169.254.169.254/32
```

### Auto-test cluster settings for CIS recommendations
```
# how to run
https://github.com/aquasecurity/kube-bench/blob/main/docs/running.md
# run on master
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest run --targets=master --version 1.22
# run on worker
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest run --targets=node --version 1.22
```

### Checksum binaries

### RBAC

### Service account mount
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: build-robot
  automountServiceAccountToken: false
  ...
```

### NodeRestriction

/etc/kubernetes/manifests/kube-apiserver.yaml add entry for - --enable-admission-plugins=NodeRestriction
and firt Node  --authorization-mode=Node

### Encrypt secrets


## Audit logs

Add params to apiserver.yaml
```
    - --audit-policy-file=/etc/kubernetes/audit/policy.yaml       # add
    - --audit-log-path=/etc/kubernetes/audit/logs/audit.log       # add
    - --audit-log-maxsize=500                                     # add
    - --audit-log-maxbackup=5                                     # add
...
  volumeMounts:
  - mountPath: /etc/kubernetes/audit      # add
    name: audit                           # add
  ...
  volumes:
  - hostPath:                               # add
      path: /etc/kubernetes/audit           # add
      type: DirectoryOrCreate               # add
    name: audit                             # add
```

