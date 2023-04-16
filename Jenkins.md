# Jenkins-in-k8s

Typical commands that might be useful when installing Jenkins on a K8s cluster (local, in my case)

***Add helm repo:***

    helm repo add jenkins https://charts.jenkins.io
	helm repo update

***Install Jenkins:***

    helm upgrade --install \
        --create-namespace -n jenkins jenkins jenkins/jenkins \
        --set controller.ingress.enabled=true \
        --set controller.ingress.hostName=jenkins.k8s.my

***Show initial admin password:***

    kubectl exec -n jenkins -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo

 ***Uninstall:***

     helm uninstall jenkins -n jenkins
