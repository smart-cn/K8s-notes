# Jenkins-in-k8s

Typical commands that might be useful when installing Jenkins on a K8s cluster (local, in my case)

***Add helm repo and create namespace:***

    helm repo add jenkins https://charts.jenkins.io
	helm repo update
	kubectl create -n jenkins

***Install Jenkins:***

    helm upgrade --install jenkins -n jenkins jenkins/jenkins

***Add ingress***

    kubectl apply -f - <<EOF
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: jenkins
      namespace: jenkins
      annotations:
        kubernetes.io/ingress.allow-http: "false"
        kubernetes.io/ingress.class: "nginx"
        nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
        nginx.ingress.kubernetes.io/ssl-redirect: "true"
        nginx.ingress.kubernetes.io/proxy-body-size: 20m
    spec:
      rules:
        - host: jenkins.k8s.my
          http:
            paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: jenkins
                    port:
                      number: 8080
    EOF

***Show initial root password:***

    kubectl exec -n jenkins -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo

 ***Uninstall:***

     helm uninstall jenkins -n jenkins
