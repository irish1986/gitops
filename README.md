# gitops

## setup host

```bash
sudo apt update && sudo apt upgrade
sudo apt install qemu-guest-agent -y
sudo apt install nfs-common open-iscsi -y
sudo reboot now
sudo modprobe iscsi_tcp
```

## Setup longhorn

1. Check node are ready

```bash
curl -sSfL https://raw.githubusercontent.com/longhorn/longhorn/v1.7.2/scripts/environment_check.sh | bash
```

## Install k3s

```bash
curl -sfL https://get.k3s.io  | INSTALL_K3S_EXEC="--disable local-storage --disable servicelb --disable=traefik --disable-cloud-controller
--tls-san=${CLUSTER_VIP} " sh -
sudo cat /etc/rancher/k3s/k3s.yaml
```

## Kubernetes Cluster

Let's use [K3D](https://k3d.io/v5.4.6/) because it [K3S](https://k3s.io/) in docker so everything is easy.

1. Install K3D.

```bash
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

1. Init K3D small cluster (1 server, 2 nodes) using a config file.

```bash
k3d cluster create --config cluster.yml
```

1. Check if everything is OK.

```bash
kubectl get nodes
```

1. Deploy nginx manifest

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

1. Check what is deployed

```bash
kubectl describe pod nginx
kubectl port-forward service/nginx 8080:80
```

## FluxCD

1. Install FluxCD CLI

```bash
        curl -s https://fluxcd.io/install.sh | sudo bash
```

1. Create token [here](https://github.com/settings/tokens).

```bash
        export GITHUB_TOKEN=<your-token>
        export GITHUB_USER=irish1986
        export GITHUB_REPO=gitops
```

1. Create a repo

```bash
        mkdir ${GITHUB_REPO}
        cd ${GITHUB_REPO}
        gh repo create ${GITHUB_REPO} --public
        git init
        touch README
        git add .
        git commit -m "first commit"
        git branch -M main
        git remote add origin https://github.com/irish1986/gitops-demo.git
        git push -u origin main
```

1. Bootstrap the repo.

```bash
flux bootstrap github \
--owner=${GITHUB_USER} \
--repository=${GITHUB_REPO} \
--branch=${CLUSTER_BRANCH} \
--path=clusters/${CLUSTER_ENV} \
--personal \
--private=false \
--token-auth
```

1. Deploy nginx manifest via FluxCD.  Move manifest under FluxCD directory.

1. Deploy nginx manifest via FluxCD

```bash
        git add .
        git commit -m "something something"
        git push origin main
        flux get kustomizations --watch
        flux reconcile kustomization flux-system --with-source
```

1. Check status

```bash
        kubectl describe pod nginx
        kubectl port-forward service/nginx 8080:80
```

1. Create ImageRepository

```bash
        flux create image repository nginx \
        --image=nginx \
        --interval=1m \
        --export > ./bootstrap/default/manifest/repo.yaml
```

1. Create ImagePolicy

```bash
        flux create image policy nginx \
        --image-ref=nginx \
        --select-semver=1.23.x \
        --export > ./bootstrap/default/manifest/policy.yaml
```

1. Deploy repo and policy

```bash
        git add .
        git commit -m "something something"
        git push origin main
        flux get kustomizations --watch
        flux reconcile kustomization flux-system --with-source
```

1. Enable policy on deployment.yaml manifest

```bash
        spec:
        containers:
        - name: nginx
            image: nginx:1.23.0  # {"$imagepolicy": "flux-system:nginx"}
```

1. Create ImageUpdateAutomation (WARRRRRPPP SPEEEEDDD !!!)

```bash
        flux create image update flux-system \
        --git-repo-ref=flux-system \
        --git-repo-path="./bootstrap/default" \
        --checkout-branch=main \
        --push-branch=main \
        --author-name=fluxcdbot \
        --author-email=fluxcdbot@users.noreply.github.com \
        --commit-template="" \
        --export > ./bootstrap/flux-system-automation.yaml
```

1. Deploy automation

```bash
        git add .
        git commit -m "something something"
        git push origin main
        flux get kustomizations --watch
        flux reconcile kustomization flux-system --with-source

        GO TO GITHUB, WAIT FOR A MINUTE.
```

1. Check status

```bash
        git pull
        kubectl describe pod nginx
```

1. Break stuff !

```bash
        KUBE_EDITOR=nano kubectl edit deployment nginx-deployment
        kubectl describe pod nginx
        flux get kustomizations --watch
        flux reconcile kustomization flux-system --with-source
```

## Clean up

Delete local cluster

```bash
    k3d cluster delete k3d-cluster
```
