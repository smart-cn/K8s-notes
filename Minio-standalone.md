# Minio(standalone mode)-in-k8s

Typical commands that might be useful when installing Minio on a K8s cluster (local, in my case)

***Add helm repo:***

    helm repo add minio https://charts.min.io/
    helm repo update

***Install Minio:***

    helm upgrade --install \
        --create-namespace -n minio minio minio/minio \
        --set consoleIngress.enabled=true \
        --set consoleIngress.hosts[0]=minio.k8s.my \
        --set mode=standalone \
        --set persistence.size=50Gi \
        --set resources.requests.memory=1Gi

***Show initial credentials:***

    echo "Login: $(kubectl get secret minio -n minio -ojsonpath='{.data.rootUser}' | base64 --decode ; echo)" && echo "Password: $(kubectl get secret minio -n minio -ojsonpath='{.data.rootPassword}' | base64 --decode ; echo)"

 ***Uninstall:***

     helm uninstall minio -n minio
