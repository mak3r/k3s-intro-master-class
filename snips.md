# Server
```
curl -sfL https://get.k3s.io | \
  INSTALL_K3S_EXEC=" \
  --write-kubeconfig-mode 644 \
  --datastore-endpoint mysql://k3s-admin:k3s-admin-pw@tcp(mak3r:3306)/k3sdb -t agent-secret \
  --tls-san mak3r.lan \
  --node-taint k3s-controlplane=true:NoExecute" \
  INSTALL_K3S_VERSION="v1.0.0" 
  sh -
```

# Agent
```
curl -sfL https://get.k3s.io | \
  INSTALL_K3S_EXEC="\
  agent \
  -t agent-secret \
  --server https://mak3r.lan:6443" \
  INSTALL_K3S_VERSION="v1.0.0" \
  sh -
```