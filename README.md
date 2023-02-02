# gitops

## Kubernetes Cluster

Let's use [K3D](https://k3d.io/v5.4.6/) because it [K3S](https://k3s.io/) in docker so everything is easy.

1. Install K3D.

        wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

1. Init K3D small cluster (1 server, 2 nodes) using a config file.

        k3d cluster create --config cluster.yml 

1. Check if everything is OK.

        alias k="kubectl"
        kubectl get nodes

1. Deploy nginx manifest

        kubectl apply -f deployment.yaml
        kubectl apply -f service.yaml

1. Check what is deployed

        kubectl get deployment -o wide

## FluxCD

1. Install FluxCD CLI

        curl -s https://fluxcd.io/install.sh | sudo bash

1. Create token [here](https://github.com/settings/tokens).

        export GITHUB_TOKEN=<your-token>
        export GITHUB_USER=irish1986
        export GITHUB_REPO=gitops

1. Create a repo 
        
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

1. Bootstrap the repo.

        flux bootstrap github \
            --components-extra=image-reflector-controller,image-automation-controller \
            --owner=${GITHUB_USER} \
            --repository=${GITHUB_REPO} \
            --branch=main \
            --path=bootstrap \
            --personal \
            --token-auth

1. Deploy nginx manifest via FluxCD

        Move manifest under FluxCD directory.

1. Deploy nginx manifest via FluxCD

        git add .
        git commit -m "something something"
        git push origin main
        flux get kustomizations --watch
        flux reconcile kustomization flux-system --with-source

1. Check status

        kubectl describe pod nginx
        kubectl port-forward service/nginx 8080:80

1. Create ImageRepository 

        flux create image repository nginx \
        --image=nginx \
        --interval=1m \
        --export > ./bootstrap/app/repo.yaml

1. Create ImagePolicy 

        flux create image policy nginx \
        --image-ref=nginx \
        --select-semver=1.23.x \
        --export > ./bootstrap/app/policy.yaml

1. Deploy repo and policy

        git add .
        git commit -m "something something"
        git push origin main
        flux get kustomizations --watch
        flux reconcile kustomization flux-system --with-source

1. Enable policy on deployment.yaml manifest

        spec:
        containers:
        - name: nginx
            image: nginx:1.23.0  # {"$imagepolicy": "flux-system:nginx"}

1. Create ImageUpdateAutomation (WARRRRRPPP SPEEEEDDD !!!)

        flux create image update flux-system \
        --git-repo-ref=flux-system \
        --git-repo-path="./bootstrap/default" \
        --checkout-branch=main \
        --push-branch=main \
        --author-name=fluxcdbot \
        --author-email=fluxcdbot@users.noreply.github.com \
        --commit-template="" \
        --export > ./bootstrap/flux-system-automation.yaml

1. Deploy automation

        git add .
        git commit -m "something something"
        git push origin main
        flux get kustomizations --watch
        flux reconcile kustomization flux-system --with-source

        GO TO GITHUB, WAIT FOR A MINUTE.

1. Check status

        git pull
        kubectl describe pod nginx

1. Break stuff !

        KUBE_EDITOR=nano kubectl edit deployment nginx-deployment
        kubectl describe pod nginx
        flux get kustomizations --watch
        flux reconcile kustomization flux-system --with-source

## Clean up

Delete local cluster

    k3d cluster delete k3d-cluster