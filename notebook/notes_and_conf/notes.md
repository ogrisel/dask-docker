To start a minikube in VM that simulates a small cluster on my laptop:

    minikube start --cpus 12 --memory 12000 --disk-size 50g --extra-config=kubelet.MaxPods=20

To upgrade to a specific kubernetes version, append:

    --kubernetes-version=v1.10.0

You need RBAC enabled which is enabled by default when using minikube 0.26+ with kubeadm.
For kubernetes < 1.10.0 you need to make it explicit:

    --extra-config=apiserver.Authorization.Mode=RBAC

and after 1.10.0 with localkube installs (non-default) you need to pass:

    --extra-config=apiserver.authorization-mode=RBAC


To setup the cluster-admin role binding used by helm / tiller:

    kubectl create clusterrolebinding tiller --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

To setup an admin level role binding for dask-kubernetes

    kubectl --namespace jupyterhub create serviceaccount dask-user
    kubectl create clusterrolebinding dask-user --clusterrole cluster-admin --serviceaccount=jupyterhub:dask-user


Public URL of the reverse proxy:

    minikube service --url -n jupyterhub proxy-public


To speed up image update on minikube:

https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615


To build the custom image locally directly on the minikube node, without having
to go through docker hub:

    eval $(minikube docker-env)
    docker build -t ogrisel/dask-notebook-extended:latest .

Alternatively it is possible to setup a local registry in the kubernetes cluster:

Local registry conf from the coco98/kube-registry.yaml gist:

    wget https://gist.github.com/coco98/b750b3debc6d517308596c248daf3bb1/raw/6efc11eb8c2dce167ba0a5e557833cc4ff38fa7c/kube-registry.yaml

    kubectl create -f kube-registry.yaml

    kubectl port-forward --namespace kube-system \
    $(kubectl get po -n kube-system | grep kube-registry-v0 | \
    awk '{print $1;}') 5000:5000

    docker build -t localhost:5000/localimage:tag .
    docker push localhost:5000/localimage:tag
