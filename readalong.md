# Intro to k3s slide notes

## Master components provide the cluster’s control plane.
* api-server: frontend for the control plane
* etcd: cluster data store
* kube-scheduler: selects nodes for newly created pods to run on (hardware/software/policy contstraints)
* kube-controller-manager: single process of the default controllers
* cloud-controller-manager: cloud provider specific control loops

## Node components
* kubelet: node agent insures containers are running inside a pod
    * Pod - smallest deployable units that can be created and managed in k8s
* kube-proxy: maintains network rules on the node - implementing part of the K8s "Service" concept
* Container runtime



