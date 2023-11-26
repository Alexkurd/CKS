# CKS

### Tips after passing exam

The main problem is time. So check which questions can you do faster and choose tools according.
Check beforehand allowed manuals to copy-paste templates(E.g. secrets in pods, networkpolicy, apparmor).
Scroll in browser is lagging, so search the words.
When you want to flag the question `save result to file` for later you should at least create the output file.
Holding arrow buttons is just one click.
Nano maybe easier for most questions, but `Ctrl+W` or `Ctrl+Alt+W` (Search text) is not working. 
Vi is default editor for `k edit resource`. Use `dd` to delete string. Use `I` instead of `Insert` to change mode.
Vi wraps strings by default, so it's usage for falco rules is better.
Trivy version is old(~0.19), so `trivy k8s pods --report summary -n kube-system` is not working.

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

## Open policy agent(OPA)
Gatekeeper added as CRD on AdmissionController level.
Create Kind.ConstraintTemplate name: k8strustedimages and add kind: K8sTrustedImages
Template should describe general conditions without targets using rego syntax.

### Trusted images

```
 violation[{"msg": msg}] {
          image := input.review.object.spec.containers[_].image
          not startswith(image, "docker.io/")
          not startswith(image, "k8s.gcr.io/")
          msg := "not trusted image!"
        }
```
### Required labels
```
 violation[{"msg": msg, "details": {"missing_labels": missing}}] {
          provided := {label | input.review.object.metadata.labels[label]}
          required := {label | label := input.parameters.labels[_]}
          missing := required - provided
          count(missing) > 0
          msg := sprintf("you must provide labels: %v", [missing])
        }
```
### Minimal replica count
```
violation[{"msg": msg, "details": {"missing_replicas": missing}}] {
          provided := input.review.object.spec.replicas
          required := input.parameters.min
          missing := required - provided
          missing > 0
          msg := sprintf("you must provide %v more replicas", [missing])
        }
```

## Supply chain

### Image footprint

- Make small image with minimal set of apps. Use multi-stage builds.
- Set image versions. Don't use latest.
- Set noroot user for app. // RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser && USER appuser
- Make readonly filesystem. // RUN chmod a-w /etc
- No shell access. // RUN rm -rf /bin/*
### Vulnerability scan
Clair
AIO deployment with api.

Trivy
```
docker run ghcr.io/aquasecurity/trivy:latest image nginx:latest
```
### Static analysis

OPA/Conftest
```
docker run --rm -v $(pwd):/project openpolicyagent/conftest test Dockerfile --all-namespaces
```
```
docker run --rm -v $(pwd):/project openpolicyagent/conftest test deploy.yaml
```

Kubesec
```
docker run -i kubesec/kubesec:512c5e0 scan /dev/stdin < pod.yaml
```

## Runtime security

### Strace

strace curl //create new process

strace -p pid //attach to the process

strace -p 528 -cw //collect summary

### Falco

/etc/falco/rules.d

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

