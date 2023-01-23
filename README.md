# gitops

## Kubernetes Cluster

Let's use [K3D](https://k3d.io/v5.4.6/) because it [K3S](https://k3s.io/) in docker so everything is easy.

Install K3D.

    wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

Init K3D small cluster (1 server, 2 nodes) using a config file.

     k3d cluster create --config cluster.yml 

Check if everything is OK.

    kubectl get nodes
    kubectl get all

## FluxCD

Install FluxCD CLI

    curl -s https://fluxcd.io/install.sh | sudo bash

Create a token [here](https://github.com/settings/tokens).

Bootstrap repo

    flux bootstrap github \
      --components-extra=image-reflector-controller,image-automation-controller \
      --owner=irish1986 \
      --repository=gitops \
      --branch=main \
      --path=bootstrap \
      --personal \
      --token-auth

    flux reconcile kustomization flux-system --with-source


## Clean up

Delete local cluster

    k3d cluster delete k3d-cluster