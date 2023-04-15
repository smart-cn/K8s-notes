# Gitlab-in-k8s

Typical commands that might be useful when installing Gitlab on a K8s cluster (local, in my case)

***Add helm repo and create namespace:***

    helm repo add gitlab https://charts.gitlab.io/
    helm repo update  
    helm search repo gitlab/gitlab -l

***Install Gitlab** (required PstgreSQL version you can find here - https://docs.gitlab.com/charts/installation/deployment.html. This command - is for Gitlab **v. 15.10.2**):*

    helm upgrade --install gitlab --create-namespace -n gitlab gitlab/gitlab \
     --version 6.10.2 \
     --timeout 600s \
     --set global.hosts.domain=k8s.my \
     --set global.hosts.kas.name=kas-gitlab.k8s.my \
     --set global.hosts.minio.name=minio-gitlab.k8s.my \
     --set global.hosts.registry.name=registry-gitlab.k8s.my \
     --set global.ingress.tls.enabled=false \
     --set global.hosts.https=false \
     --set certmanager-issuer.email=admin@k8s.my \
     --set postgresql.image.tag=13.6.0 \
     --set global.edition=ce

***Show initial root password:***

    kubectl get secret gitlab-gitlab-initial-root-password -n gitlab -ojsonpath='{.data.password}' | base64 --decode ; echo

 ***Uninstall:***

     helm uninstall gitlab -n gitlab
